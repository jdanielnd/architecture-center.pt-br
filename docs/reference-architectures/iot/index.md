---
title: Arquitetura de referência de IoT do Azure
description: Arquitetura recomendada para aplicativos de IoT no Azure usando os componentes do PaaS (plataforma como serviço)
titleSuffix: Azure Reference Architectures
author: MikeWasson
ms.date: 01/09/2019
---

# <a name="azure-iot-reference-architecture"></a>Arquitetura de referência de IoT do Azure

Essa arquitetura de referência mostra uma arquitetura recomendada para aplicativos de IoT no Azure usando os componentes do PaaS (plataforma como serviço).

![Diagrama da arquitetura](./_images/iot.png)

Os aplicativos de IoT podem ser descritos como **coisas** (dispositivos) enviando dados que geram **insights**. Esses insights geram **ações** para melhorar um negócio ou um processo. Um exemplo é um mecanismo (a coisa) enviando dados de temperatura. Esses dados são usados para avaliar se o mecanismo está funcionando conforme o esperado (o insight). O insight é usado para priorizar proativamente o agendamento de manutenção para o mecanismo (a ação).

Essa arquitetura de referência usa componentes do PaaS do Azure (plataforma como serviço). Outras opções para criar soluções de IoT no Azure incluem:

- [Azure IoT Central](/azure/iot-central/). IoT Central é uma solução de SaaS (software como serviço) totalmente gerenciada. Ele abstrai as escolhas técnicas e permite que você se concentre exclusivamente em sua solução. Essa simplicidade vem com uma compensação por ser menos personalizável do que uma solução baseada em PaaS.
- Usando componentes do OSS, como a pilha SMACK (Spark, Mesos, Akka, Cassandra, Kafka) implantada nas VMs do Azure. Esta abordagem oferece uma grande quantidade de controle, mas é mais complexa.

Em um nível alto, há duas maneiras de processar dados de telemetria, o caminho quente e o caminho frio. A diferença tem a ver com os requisitos de latência e acesso a dados.

- O **caminho quente** analisa os dados quase em tempo real, conforme chegam. No caminho quente, a telemetria deve ser processada com latência muito baixa. O caminho quente geralmente é implementado usando um mecanismo de processamento de fluxo. A saída pode disparar um alerta ou ser gravada em um formato estruturado que pode ser consultado com ferramentas analíticas.
- O **caminho frio** executa processamento em lotes em intervalos mais longos (por hora ou diariamente). O caminho frio geralmente opera em grandes volumes de dados, mas os resultados não precisam ser tão pontuais como o caminho quente. No caminho frio, a telemetria bruta é capturada e, em seguida, inserida em um processo em lote.

## <a name="architecture"></a>Arquitetura

Essa arquitetura é formada pelos componentes a seguir. Alguns aplicativos não exigem todos os componentes listados aqui.

**Dispositivos IoT**. Os dispositivos podem se registrar com segurança na nuvem e podem se conectar à nuvem para enviar e receber dados. Alguns dispositivos podem ser **dispositivos de borda** que executam algum processamento de dados no próprio dispositivo ou em um gateway de campo. É recomendável o uso do [Azure IoT Edge](/azure/iot-edge/) para o processamento de borda.

**Gateway de Nuvem**. Um gateway de nuvem fornece um hub de nuvem para que os dispositivos se conectem com segurança à nuvem e enviem dados. Ele também fornece gerenciamento de dispositivos, recursos, incluindo o comando e o controle de dispositivos. Para o gateway de nuvem, recomendamos o [Hub IoT](/azure/iot-hub/). O Hub IoT é um serviço de nuvem hospedado que recebe eventos dos dispositivos, atuando como um agente de mensagem entre os dispositivos e os serviços de back-end. O Hub IoT fornece conectividade segura, ingestão de eventos, comunicação bidirecional e gerenciamento de dispositivos.

**Provisionamento de dispositivos**. Para registrar e conectar grandes conjuntos de dispositivos, é recomendável usar o [Serviço de Provisionamento de Dispositivos no Hub IoT](/azure/iot-dps/) (DPS). O DPS permite que você atribua e registre dispositivos em pontos de extremidade específicos do Hub IoT do Azure em escala.

**Processamento de fluxo**. O processamento de fluxo analisa grandes fluxos de registros de dados e avalia as regras para eles. Para o processamento de fluxo, recomendamos o [Azure Stream Analytics](/azure/stream-analytics/). O Stream Analytics pode executar análises complexas em escala, usando as funções de janela de tempo, agregações de fluxo e junções de fontes de dados externas. Outra opção é o Apache Spark no [Azure Databricks](/azure/azure-databricks/).

O aprendizado de máquina permite que os algoritmos de previsão sejam executados em dados de telemetria históricos, habilitando cenários como a manutenção preditiva. Para o aprendizado de máquina, é recomendável o [Serviço do Azure Machine Learning](/azure/machine-learning/service/).

O **armazenamento de caminho quente** contém dados que devem estar disponíveis imediatamente no dispositivo de relatórios e visualização. Para o armazenamento de caminho quente, recomendamos o [Cosmos DB](/azure/cosmos-db/introduction). O Cosmos DB é um banco de dados multimodelo distribuído globalmente.

O **armazenamento de caminho frio** contém dados que são mantidos a longo prazo e são usados para processamento em lotes. Para o armazenamento de caminho frio, é recomendável o [Armazenamento de Blobs do Azure](/azure/storage/blobs/storage-blobs-introduction). Os dados podem ser arquivados no armazenamento de Blobs indefinidamente com baixo custo e são facilmente acessíveis para processamento em lotes.

A **Transformação de dados** manipula ou agrega o fluxo de telemetria. Os exemplos incluem a transformação de protocolo, como converter dados binários para JSON ou combinar pontos de dados. Se os dados devem ser transformados antes de chegar ao Hub IoT, é recomendável usar um [gateway de protocolo](/azure/iot-hub/iot-hub-protocol-gateway) (não mostrado). Caso contrário, os dados podem ser transformados depois de chegar ao Hub IoT. Nesse caso, recomendamos usar o [Azure Functions](/azure/azure-functions/). As funções têm integração interna com o Hub IoT, Cosmos DB e com o Armazenamento de Blobs.

A **Integração de processos de negócios** executa ações com base nas informações sobre os dados do dispositivo. Isso pode incluir armazenar mensagens informativas, gerar alarmes, envio de email ou mensagens SMS ou a integração com o CRM. É recomendável usar os [Aplicativos Lógicos do Azure](/azure/logic-apps/logic-apps-overview) para a integração de processos de negócios.

O **Gerenciamento de usuários** restringe quais usuários ou grupos podem executar ações nos dispositivos, como a atualização do firmware. Ele também define os recursos para os usuários nos aplicativos. Recomendamos usar o [Azure Active Directory](/azure/active-directory/) para autenticar e autorizar usuários.

## <a name="scalability-considerations"></a>Considerações sobre escalabilidade

Um aplicativo de IoT deve ser criado como serviços discretos que podem ser dimensionados independentemente. Considere os seguintes pontos de escalabilidade:

**Hub IoT**. Para o Hub IoT, considere os seguintes fatores de dimensionamento:

- A [cota diária](/azure/iot-hub/iot-hub-devguide-quotas-throttling) máxima de mensagens no Hub IoT.
- A cota de dispositivos conectados em uma instância do Hub IoT.
- Taxa de transferência de ingestão &mdash; a rapidez com que o Hub IoT pode receber as mensagens.
- Taxa de transferência de processamento &mdash; a rapidez com que as mensagens de entrada são processadas.

Cada Hub IoT é provisionado com um determinado número de unidades em uma camada específica. A camada e o número de unidades determinam a cota diária máxima de mensagens que os dispositivos podem enviar para o hub. Para obter mais informações, confira cotas e limitação do Hub IoT. Você pode escalar verticalmente um hub sem interromper as operações existentes.

**Stream Analytics**. Os trabalhos do Stream Analytics são melhor dimensionados se forem paralelos em todos os pontos no pipeline do Stream Analytics, desde a entrada para a consulta até a saída. Um trabalho totalmente paralelo permite que o Stream Analytics divida o trabalho em vários nós de computação. Caso contrário, o Stream Analytics tem que combinar os dados de fluxo em um só lugar. Para obter mais informações, confira [Aproveitar a paralelização de consultas no Azure Stream Analytics](/azure/stream-analytics/stream-analytics-parallelization).

O Hub IoT particiona automaticamente mensagens de dispositivo com base na ID do dispositivo. Todas as mensagens de um determinado dispositivo sempre chegarão na mesma partição, mas uma única partição terá mensagens de vários dispositivos. Portanto, a unidade da paralelização é a ID da partição.

**Functions**. Ao ler a partir do ponto de extremidade dos Hubs de Eventos, há um máximo de instâncias de função por partição do hub de eventos. A taxa máxima de processamento é determinada pela rapidez com que uma instância de função pode processar os eventos de uma única partição. A função deve processar mensagens em lotes.

**Cosmos DB**. Para expandir uma coleção do Cosmos DB, crie a coleção com uma chave de partição e inclua a chave de partição em cada documento que você escreve. Para obter mais informações, confira [Melhores práticas para a escolha de uma chave de partição](/azure/cosmos-db/partition-data#best-practices-when-choosing-a-partition-key).

- Se você armazenar e atualizar um único documento por dispositivo, a ID do dispositivo é uma boa chave de partição. As gravações são distribuídas uniformemente entre as chaves. O tamanho de cada partição é estritamente limitado, porque há um único documento para cada valor de chave.
- Se você armazenar um documento separado para cada mensagem do dispositivo, usando a ID do dispositivo como uma chave de partição, rapidamente excede o limite de 10 GB por partição. A ID da mensagem é uma chave de partição melhor nesse caso. Normalmente, você incluiria ainda a identificação do dispositivo no documento para indexação e consulta.

## <a name="security-considerations"></a>Considerações de segurança

### <a name="trustworthy-and-secure-communication"></a>Comunicação confiável e segura

Todas as informações recebidas e enviadas para um dispositivo devem ser confiáveis. A menos que um dispositivo tenha suporte para os recursos de criptografia a seguir, ele deve ser restrito a redes locais e toda a comunicação entre redes deve passar por um gateway de campo:

- Criptografia de dados com um algoritmo de criptografia de chave simétrica provavelmente seguro, publicamente analisado e amplamente implementado.
- Assinatura digital com um algoritmo de assinatura de chave simétrica provavelmente seguro, publicamente analisado e amplamente implementado.
- Suporte para o TLS 1.2 para TCP ou outros caminhos de comunicação baseada em fluxo ou DTLS 1.2 para caminhos de comunicação baseada em datagrama. Suporte de manipulação de certificado X.509 é opcional e pode ser substituído pelo modo de chave pré-compartilhada mais eficiente de computação e eficiente de transmissão para o TLS, que pode ser implementado com suporte para os algoritmos AES e SHA-2.
- Repositório de chaves atualizável e chaves por dispositivo. Cada dispositivo deve ter o material de chave exclusiva ou tokens que o identifiquem perante o sistema. Os dispositivos devem armazenar a chave com segurança (por exemplo, usando um repositório de chaves seguro). O dispositivo deve ser capaz de atualizar as chaves ou tokens periodicamente, ou de modo reativo em situações de emergência, como uma violação de sistema.
- O software de aplicativo e o firmware no dispositivo devem permitir atualizações para habilitar o reparo de vulnerabilidades de segurança descobertas.

No entanto, muitos dispositivos são muito restritos para dar suporte a esses requisitos. Nesse caso, um gateway de campo deve ser usado. Os dispositivos se conectam com segurança ao gateway de campo por meio de uma rede local e o gateway permite a comunicação segura com a nuvem.

### <a name="physical-tamper-proofing"></a>À prova de violação física

É altamente recomendável que o design do dispositivo incorpore recursos que protejam contra tentativas de manipulação física, para ajudar a garantir a integridade de segurança e a confiabilidade do sistema geral.

Por exemplo: 

- Escolha microcontroladores/microprocessadores ou hardware auxiliar que forneçam armazenamento seguro e uso do material de chave de criptografia, como a integração de TPM (módulo de plataforma confiável).
- Proteja o carregador de inicialização e o carregamento de software, ancorados no TPM.
- Use sensores para detectar tentativas de invasão e tentativas de manipular o ambiente do dispositivo com alertas e potencialmente “autodestruição digital” do dispositivo.

Para outras considerações de segurança, confira [Arquitetura de segurança da Internet das Coisas (IoT)](/azure/iot-fundamentals/iot-security-architecture).

### <a name="monitoring-and-logging"></a>Monitoramento e registro em log

Sistemas de monitoramento e registro em log são usados para determinar se a solução está funcionando e para ajudar a solucionar problemas. Os sistemas de monitoramento e registro em log ajudam a responder às seguintes perguntas operacionais:

- Os dispositivos ou sistemas estão em uma condição de erro?
- Os dispositivos ou sistemas estão configurados corretamente?
- Os dispositivos ou sistemas estão gerando dados precisos?
- Os sistemas estão atendendo às expectativas dos clientes finais e de negócios?

As ferramentas de monitoramento e registro em log são geralmente compostas pelos quatro componentes a seguir:

- Ferramentas de visualização da linha do tempo e de desempenho do sistema para o monitoramento do sistema e para a solução básica de problemas.
- Ingestão de dados armazenados em buffer, para armazenar em buffer dados de log.
- Armazenamento de persistência, para armazenar dados de log.
- Recursos de pesquisa e consulta, para exibir dados de log para uso na solução detalhada de problemas.

Os sistemas de monitoramento fornecem insights sobre a integridade, a segurança, a estabilidade e o desempenho de uma solução de IoT. Esses sistemas também podem fornecer uma exibição mais detalhada, registrar alterações de configuração de componentes e fornecer dados de log extraídos, que podem mostrar possíveis vulnerabilidades de segurança, aprimorar o processo de gerenciamento de incidentes e ajudar o proprietário do sistema a solucionar problemas. Soluções abrangentes de monitoramento incluem a capacidade de consultar informações de subsistemas específicos ou de agregação nos vários subsistemas.

O desenvolvimento do sistema de monitoramento deve começar com a definição de operação íntegra, conformidade a normas e requisitos de auditoria. As métricas coletadas podem incluir:

- Dispositivos físicos, dispositivos de borda e componentes de infraestrutura que relatam as alterações de configuração.
- Aplicativos que relatam alterações de configuração, logs de auditoria de segurança, taxas de solicitação, tempos de resposta, taxas de erro e estatísticas de coleta de lixo para linguagens gerenciadas.
- Bancos de dados, armazenamentos de persistência e caches que relatam desempenho de consultas e gravação, alterações de esquema, log de auditoria de segurança, bloqueios ou deadlocks, desempenho do índice, CPU, memória e uso do disco.
- Serviços gerenciados (IaaS, PaaS, SaaS e FaaS) que relatam as métricas de integridade e as alterações de configuração que afetam o desempenho e a integridade do sistema dependente.

A visualização da métrica de monitoramento alerta os operadores para instabilidades do sistema e facilita a resposta a incidentes.

### <a name="tracing-telemetry"></a>Telemetria de rastreamento

A telemetria de rastreamento permite que um operador acompanhe a jornada de um trecho da telemetria desde a criação por meio do sistema. O rastreamento é importante para depuração e solução de problemas. Para soluções de IoT que usam o Hub IoT do Azure e o [SDKs do dispositivo de Hub IoT](/azure/iot-hub/iot-hub-devguide-sdks), os datagramas de rastreamento podem ser criados como mensagens da nuvem para dispositivo e incluídos no fluxo de telemetria.

### <a name="logging"></a>Registro em log

Os sistemas de registro são essenciais para entender quais ações ou atividades uma solução realizou, falhas que ocorreram e que podem fornecer ajuda na correção dessas falhas. Os logs podem ser analisados para ajudar a entender e corrigir condições de erro, aprimorar as características de desempenho e garantir a conformidade com as regras e regulamentos vigentes.

Embora o registro em log de texto sem formatação tenha um impacto menor nos custos de desenvolvimento iniciais, é mais difícil para um computador ler/analisar. É recomendável que o registro em log estruturado seja usado, porque as informações coletadas são analisáveis por máquina e legíveis por humanos. O registro em log estruturado adiciona contexto e metadados situacionais às informações de log. No registro em log estruturado, as propriedades são cidadãos de primeira classe formatados como pares de chave/valor, ou com um esquema fixo, para aprimorar os recursos de pesquisa e consulta.

## <a name="next-steps"></a>Próximas etapas

- Para obter uma discussão mais detalhada das opções de arquitetura e implementação recomendadas, confira [Arquitetura de Referência do IoT do Microsoft Azure](http://aka.ms/iotrefarchitecture) (PDF).

- Para obter a documentação detalhada dos vários serviços de IoT do Azure, confira [Conceitos básicos do IoT do Azure](/azure/iot-fundamentals/).

- Há uma amostra da implementação de IoT no [GitHub](https://github.com/mspnp/iot-guidance).
