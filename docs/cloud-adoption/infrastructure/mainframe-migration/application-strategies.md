---
title: 'Migração de mainframe: Migração de aplicativos de mainframe'
description: Migre aplicativos de ambientes de mainframe no Azure, uma infraestrutura escalonável, altamente disponível e comprovada para sistemas que atualmente executam em mainframes
author: njray
ms.date: 12/26/2018
ms.openlocfilehash: 2a22eb038da693671ce309c76afcfc41946034f3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246387"
---
# <a name="mainframe-application-migration"></a>Migração de aplicativos de mainframe

Ao migrar aplicativos de ambientes de mainframe no Azure, a maioria das equipes segue uma abordagem pragmática: reutilizar onde e quando possível, e, em seguida, iniciar implantação em fases, em que os aplicativos sejam reconfigurados ou substituídos.

Migração de aplicativos normalmente envolvem uma ou mais das seguintes estratégias:

- Hospedar novamente: Você pode mover o código existente, programas e aplicativos de mainframe e, em seguida, recompilar o código que será executado em um emulador de mainframe hospedado em uma instância de nuvem. Essa abordagem normalmente inicia com aplicativos de movimento para um emulador baseado em nuvem e, em seguida, migrar o banco de dados para um banco de dados baseado em nuvem. Alguma engenharia e refatoração são necessárias, juntamente com conversões de dados e arquivo.

    Como alternativa, você pode hospedar novamente usando um provedor de hospedagem tradicional. Um dos principais benefícios da nuvem é terceirizar o gerenciamento da infraestrutura. Você pode encontrar um provedor de datacenter que irá hospedar suas cargas de trabalho de mainframe para você. Esse modelo pode comprar o tempo, reduzir o bloqueio de fornecedor e produzir economia de custo provisório.

- Desativar: Todos os aplicativos que não são mais necessários devem ser desativados antes da migração.

- Recompilar: Algumas organizações optam por reescrever totalmente programas usando técnicas modernas. Dado o custo e a complexidade dessa abordagem, não é tão comum quanto uma abordagem lift-and-shift. Muitas vezes após esse tipo de migração, faz sentido começar substituindo módulos e o código usando mecanismos de transformação de código.

- Substitua: Essa abordagem substitui a funcionalidade de mainframe com os recursos equivalentes na nuvem. Software como serviço (SaaS) é uma opção que está usando uma solução criada especificamente para uma preocupação de empresa, como finanças, recursos humanos, fabricação ou planejamento de recursos empresariais. Além disso, muitos aplicativos específicos do setor agora estão disponíveis para resolver problemas que as soluções de mainframe personalizadas costumavam resolver anteriormente.

Você deve considerar começando pelo planejamento dessas cargas de trabalho que você deseja migrar inicialmente e, em seguida, determinar esses requisitos para a movimentação de aplicativos associados, bases de código herdado e bancos de dados.

## <a name="mainframe-emulation-in-azure"></a>Emulação de mainframe no Azure

Os serviços de nuvem do Azure podem emular os ambientes de mainframe tradicionais, permitindo a reutilização de código existente do mainframe e aplicativos. Componentes de servidor comuns que você pode emular incluem transação online (OLTP), lotes e sistemas de ingestão de dados.

### <a name="oltp-systems"></a>Sistemas OLTP

Muitos mainframes têm sistemas OLTP que processam milhares ou milhões de atualizações para grandes números de usuários. Esses aplicativos geralmente usam o processamento de transações e o software de manipulação de formulário de tela, como sistema de controle de informações do cliente (CICS), sistemas de gerenciamento de informações (IMS) e processador de interface de terminal (TIP).

Ao mover aplicativos OLTP no Azure, emuladores para monitores (TP) de processamento de transações de mainframe estão disponíveis para serem executado como infraestrutura como serviço (IaaS) usando máquinas virtuais (VMs) no Azure. A funcionalidade de tratamento e a funcionalidade de formulário também podem ser implementados pelo servidores web. Essa abordagem pode ser combinada com as APIs de banco de dados, como o objeto de dados do ActiveX (ADO), conectividade de banco de dados aberto (ODBC) e conectividade de banco de dados Java (JDBC) para acesso a dados e transações.

### <a name="time-constrained-batch-updates"></a>Atualizações em lotes de restrição de tempo

Muitos sistemas de mainframe executam atualizações mensais ou anuais de milhões de registros de conta, como aqueles usados em serviços bancários, seguros e governamentais. Mainframes lidam com esses tipos de cargas de trabalho, oferecendo a sistemas de manipulação de dados de alta taxa de transferência. Trabalhos em lotes de mainframes são normalmente seriais por natureza e dependem das operações de entrada/saída por segundo (IOPS) fornecidas pelo backbone do mainframe para desempenho.

Ambientes de lote baseados em nuvem usam a computação paralela e redes de alta velocidade para o desempenho. Se você precisa o desempenho do lote, o Azure fornece várias opções de rede, armazenamento e computação.

### <a name="data-ingestion-systems"></a>Sistemas de ingestão de dados

Os mainframes ingerem lotes grandes de dados de varejo, serviços financeiros, fabricação e outras soluções para o processamento. Com o Azure, você pode usar os utilitários de linha de comando simples, como [AzCopy](/azure/storage/common/storage-use-azcopy) para cópia de dados de local de armazenamento. Você também pode usar o serviço do [Azure Data Factory](/azure/data-factory/introduction), permitindo a ingestão de dados de armazenamentos de dados diferentes para criar e agendar fluxos de trabalho orientados a dados.

Além dos ambientes de emulação, o Azure fornece plataforma como serviço (PaaS) e análise de serviços que podem melhorar os ambientes existentes do mainframe.

## <a name="migrate-oltp-workloads-to-azure"></a>Migre cargas de trabalho OLTP para o Azure

A abordagem de lift-and-shift é a opção sem código para migrar rapidamente aplicativos existentes no Azure. Cada aplicativo é migrado “no estado em que se encontra”, o que oferece os benefícios da nuvem sem os riscos ou custos de realização de alterações no código. Usar um emulador para monitores de processamento de transação de mainframe (TP) no Azur com suporte a essa abordagem.

Os monitores TP estão disponíveis de vários fornecedores e executados em máquinas virtuais, uma infraestrutura como uma opção de serviço (IaaS) no Azure. Os diagramas a seguir de antes e depois mostram uma migração de um aplicativo online apoiado por IBM DB2, um sistema de gerenciamento de banco de dados relacionais (DBMS) em um mainframe IBM z/OS. DB2 para z/OS usa arquivos de método de acesso de armazenamento (VSAM) para armazenar os dados e Método de Acesso Sequencial Indexado (ISAM) para arquivos simples. Essa arquitetura também usa CICS para monitoramento de transação.

![Lift-and-shift de um ambiente de mainframe no Azure usando o software de emulação](../../_images/mainframe-migration/mainframe-vs-azure.png)

No Azure, ambientes de emulação são usados para executar o Gerenciador de TP e os trabalhos em lotes que usam JCL. Na camada de dados, o DB2 é substituído pelo [Banco de Dados SQL do Azure](/azure/sql-database/sql-database-technical-overview), embora o banco de dados Oracle, DB2 LUW ou Microsoft SQL Server também possa ser usado. Um emulador dá suporte a IMS, VSAM e SEQ. Ferramentas de gerenciamento do sistema de mainframe são substituídas por serviços do Azure e o software de outros fornecedores, que são executados em máquinas virtuais.

O manuseio de tela e a funcionalidade de entrada de formulário são comumente implementados usando servidores da web, que podem ser combinados com APIs de banco de dados, como ADO, ODBC e JDBC para acesso a dados e transações. A linha exata de componentes de IaaS do Azure para usar depende do sistema operacional que você preferir. Por exemplo: 

- VMs com base no Windows: Internet Information Server (IIS), juntamente com o ASP.NET para o tratamento de tela e lógica de negócios. Use o ADO.NET para acesso a dados e transações.

- VMs com base no Linux: Os servidores de aplicativos baseados em Java que estão disponíveis, como o Apache Tomcat para tratamento de tela e a funcionalidade de negócios baseado em Java. Use o JDBC para transações e acesso a dados.

## <a name="migrate-batch-workloads-to-azure"></a>Migre cargas de trabalho de lote para o Azure

Operações em lote no Azure são diferentes do ambiente de lote típico em mainframes. Trabalhos em lotes de mainframes são normalmente seriais por natureza e dependem dos IOPS fornecidos pelo backbone de mainframe por desempenho. Ambientes de lote baseados em nuvem usam a computação paralela e redes de alta velocidade para o desempenho.

Para otimizar o desempenho do lote usando o Azure, considere as opções de [computar](/azure/virtual-machines/windows/overview), [armazenar](/azure/storage/blobs/storage-blobs-introduction), [rede](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/), e [monitoramento](/azure/azure-monitor/overview) da seguinte maneira.

### <a name="compute"></a>Computação

Use:

- VMs com a velocidade de relógio mais alta. Aplicativos de mainframe são geralmente de thread único e CPUs de mainframe têm um relógio de velocidade muito alta.

- VMs com capacidade de memória grande para permitir o armazenamento em cache de dados e áreas de trabalho do aplicativo.

- VMs com vCPUs de densidade mais alta para tirar proveito de processamento com multithread, se o aplicativo dá suporte a vários threads.

- Processamento paralelo, como Azure facilmente pode ser dimensionado para processamento paralelo, oferecendo mais poder de computação para um execução de lote.

### <a name="storage"></a>Armazenamento

Use:

- [Azure SSD Premium](/azure/virtual-machines/windows/premium-storage) ou [SSD Ultra do Azure](/azure/virtual-machines/windows/disks-ultra-ssd) para máximo de IOPS disponível.

- Particionamento com vários discos para mais IOPS por tamanho de armazenamento.

- O particionamento para armazenamento para distribuir E/S sobre vários dispositivos de armazenamento de Azure.

### <a name="networking"></a>Rede

- Use [Rede Acelerada do Azure](/azure/virtual-network/create-vm-accelerated-networking-powershell) para minimizar a latência.

### <a name="monitoring"></a>Monitoramento

- Use ferramentas de monitoramento, [Azure Monitor](/azure/azure-monitor/overview), [Azure Application Insights](/azure/application-insights/app-insights-overview) e até mesmo os logs do Azure permitem que os administradores monitorem qualquer sobre o desempenho das execuções de lote e ajudar a eliminar gargalos.

## <a name="migrate-development-environments"></a>Migrar os ambientes de desenvolvimento

Arquitetura distribuída da nuvem se baseiam em um conjunto diferente de ferramentas de desenvolvimento que fornecem a vantagem de práticas modernas e linguagens de programação. Para facilitar essa transição, você pode usar um ambiente de desenvolvimento com outras ferramentas que são projetadas para emular os ambientes do IBM z/OS. A lista a seguir mostra as opções da Microsoft e outros fornecedores:

| Componente        | Opções do Azure                                                                                                                                  |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| z/OS             | Windows, Linux ou UNIX                                                                                                                      |
| CICS             | Serviços do Azure oferecidos pelo Micro Focus, Oracle, Software GT (Fujitsu), TmaxSoft, Raincode e dados NTT ou reescrever usando o Kubernetes |
| IMS              | Serviços do Azure oferecidos pela Micro Focus e Oracle                                                                                  |
| Assembler        | Serviços do Azure de Raincode e TmaxSoft; ou COBOL, C ou Java, ou são mapeados para funções do sistema operacional               |
| JCL              | JCL, PowerShell ou outras ferramentas de script                                                                                                   |
| COBOL            | COBOL, C ou Java                                                                                                                            |
| Natural          | Natural, COBOL, C ou Java                                                                                                                  |
| FORTRAN e PL/I | FORTRAN, PL/I, COBOL, C ou Java                                                                                                           |
| REXX e PL/I    | REXX, PowerShell ou outras ferramentas de script                                                                                                  |

## <a name="migrate-databases-and-data"></a>Migre bancos de dados e dados

A migração de aplicativo geralmente envolve a nova hospedagem da camada de dados. Você pode migrar o SQL Server, o código-fonte aberto e outros bancos de dados relacionais para soluções totalmente gerenciadas no Azure, tais como [Instância Gerenciada do Banco de Dados SQL do Azure](/azure/sql-database/sql-database-managed-instance), [Serviço de Banco de Dados do Azure para PostgreSQL](/azure/postgresql/overview), e [Banco de Dados do Azure para MySQL](/azure/mysql/overview) com [Serviço de Migração de Banco de Dados do Azure](/azure/dms/dms-overview).

Por exemplo, você pode migrar se a camada de dados de mainframe usa:

- IBM DB2 ou um banco de dados IMS, usar o banco de dados SQL do Azure, SQL Server, DB2 LUW ou banco de dados Oracle Database no Azure.

- VSAM e outros arquivos simples, use arquivos simples do Método Indexado de Acesso Sequencial (ISAM) para o SQL do Azure, SQL Server, DB2 LUW ou Oracle.

- Grupos de Data de Geração (GDGs), migrar para os arquivos no Azure que usam uma convenção de nomenclatura e extensões de nome de arquivo que fornecem funcionalidade semelhante ao GDGs.

A camada de dados do IBM inclui vários componentes principais que também devem migrar. Por exemplo, ao migrar um banco de dados, você também migrar uma coleção de dados contidos em pools, cada um contendo dbextents, que são conjuntos de dados z/OS VSAM. A migração deve incluir o diretório que identifica os locais de dados nos pools de armazenamento. Além disso, o plano de migração deve considerar o log de banco de dados, que contém um registro das operações executadas no banco de dados. Um banco de dados pode ter um, dois (duplo ou alternativo) ou quatro logs (duplo ou alternativo).

A migração de banco de dados também inclui os seguintes componentes:

- Gerenciador de banco de dados: Fornece acesso aos dados no banco de dados. O gerenciador de banco de dados é executado em sua própria partição em um ambiente de z/OS.

- Solicitante de aplicativo: Aceita solicitações de aplicativos antes de passá-los para um servidor de aplicativos.

- Adaptador de recursos online: Inclui componentes do solicitante de aplicativo para uso em transações do CICS.

- Adaptador de recursos em lote: Componentes do solicitante de aplicativo implementa para aplicativos de longe z/OS.

- SQL Interativo (ISQL): É executado como um aplicativo CICS e uma interface, permitindo que os usuários insiram instruções SQL ou comandos de operador.

- Aplicativo CICS: É executado sob o controle do CICS, usando fontes de dados e recursos disponíveis no CICS.

- Aplicativo do lote: Executa lógica de processo sem comunicação interativa com usuários para, por exemplo, produzir atualizações de dados em massa ou gerar relatórios a partir de um banco de dados.

## <a name="optimize-scale-and-throughput-for-azure"></a>Otimizar a taxa de transferência e escala para o Azure

Em termos gerais, mainframes escalam verticalmente, enquanto a nuvem pode ser dimensionada. Para otimizar a escala e a taxa de transferência de aplicativos de estilo mainframe em execução no Azure, é importante que você compreenda como mainframes podem separar e isolar aplicativos. Um mainframe z/OS usa um recurso chamado Partições Lógicas (LPARS) para isolar e gerenciar os recursos para um aplicativo específico em uma única instância.

Por exemplo, um mainframe pode usar uma partição lógica (LPAR) para uma região do CICS com programas associados de COBOL e um LPAR separado para DB2. LPARs adicionais geralmente são usados para o desenvolvimento, teste e ambientes de preparo.

No Azure, é mais comum usar VMs separadas para servirem para essa finalidade. Normalmente, as arquiteturas do Azure implantam VMs para a camada de aplicativo, um conjunto separado de VMs para a camada de dados, outro conjunto para o desenvolvimento e assim por diante. Cada camada de processamento pode ser otimizada usando o tipo mais adequado de máquinas virtuais e recursos para esse ambiente.

Além disso, cada camada também pode fornecer serviços de recuperação de desastres apropriados. Por exemplo, VMs do banco de dados e produção podem requerer uma recuperação ativa ou passiva, enquanto as VMs de desenvolvimento e teste dão suporte a uma recuperação fria.

A figura a seguir mostra uma possível implantação do Azure usando um site principal e um secundário. No site primário, as VMs de produção, pré-produção e teste são implantadas com alta disponibilidade. O site secundário é feito para backup e recuperação de desastres.

![A figura a seguir mostra uma possível implantação do Azure usando um site principal e um secundário](../../_images/mainframe-migration/migration-backup-DR.png)

## <a name="perform-a-staged-mainframe-to-azure"></a>Executar um mainframe preparado para o Azure

Migrar soluções de um mainframe para o Azure pode envolver uma migração *preparada*, na qual alguns aplicativos são movidos pela primeira vez e outros permanecem no mainframe temporariamente ou permanentemente. Normalmente, essa abordagem requer sistemas que permitem que aplicativos e bancos de dados para fins de interoperabilidade entre o mainframe e o Azure.

Um cenário comum é mover um aplicativo no Azure enquanto mantém os dados usados pelo aplicativo no mainframe. Software específico é usado para permitir que os aplicativos no Azure acessem dados de mainframe. Felizmente, uma ampla gama de soluções fornece integração entre o Azure e os ambientes existentes de mainframe, suporte para cenários híbridos e migração ao longo do tempo. Parceiros da Microsoft, fornecedores independentes de software e integradores de sistema podem ajudá-lo em seu percurso.

Uma opção é o [Microsoft Host Integration Server](/host-integration-server) (HIS), uma solução que fornece a arquitetura distribuída de banco de dados relacional (DRDA) necessária para aplicativos no Azure para acessar dados no DB2 que permanece no mainframe. Outras opções para integração com o mainframe no Azure incluem soluções da IBM, Attunity, Codit, outros fornecedores e opções de software livre.

## <a name="partner-solutions"></a>Soluções de parceiros

Se você estiver considerando uma migração de mainframe, o ecossistema de parceiros está disponível para ajudá-lo.

O Azure fornece uma infraestrutura comprovada, altamente disponível e escalonável para sistemas que atualmente executam em mainframes. Algumas cargas de trabalho podem ser migradas com relativa facilidade. Outras cargas de trabalho que dependem do software de sistema herdado, como o CICS e IMS, podem ser hospedadas novamente usando soluções de parceiros e migrados para o Azure ao longo do tempo. Independentemente da opção escolhida, a Microsoft e nossos parceiros estão disponíveis para ajudá-lo a otimizar o Azure enquanto mantém a funcionalidade de software do sistema de mainframe.

Para obter orientações detalhadas sobre como escolher uma solução de parceiro, consulte a [Aliança de Modernização de Plataforma](https://www.platformmodernization.org/pages/mainframe.aspx).

## <a name="learn-more"></a>Saiba mais

Para saber mais, consulte os recursos a seguir:

- [Introdução ao Azure](/azure)

- [Aliança de Modernização de Plataforma: Migração de mainframe](https://www.platformmodernization.org/pages/mainframe.aspx)

- [Implantar o IBM DB2 pureScale no Azure](https://azure.microsoft.com/resources/deploy-ibm-db2-purescale-on-azure)

- [Documentação do Host Integration Server (HIS)](/host-integration-server)
