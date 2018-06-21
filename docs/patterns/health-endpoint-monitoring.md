---
title: Monitoramento do Ponto de Extremidade de Integridade
description: Implemente verificações funcionais dentro de um aplicativo cujas ferramentas externas podem acessar por meio de pontos de extremidade expostos em intervalos regulares.
keywords: padrão de design
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- availability
- management-monitoring
- resiliency
ms.openlocfilehash: 3b3bce46b460148af17bfe6064cd052a5f9a6458
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847842"
---
# <a name="health-endpoint-monitoring-pattern"></a>Padrão de monitoramento do ponto de extremidade de integridade

[!INCLUDE [header](../_includes/header.md)]

Implemente verificações funcionais dentro de um aplicativo cujas ferramentas externas podem acessar por meio de pontos de extremidade expostos em intervalos regulares. Isso pode ajudar a verificar se os aplicativos e serviços estão funcionando corretamente.

## <a name="context-and-problem"></a>Contexto e problema

É uma boa prática e geralmente um requisito de negócios para monitorar serviços back-end e aplicativos Web e garantir que eles estejam disponíveis e funcionando corretamente. No entanto, é mais difícil monitorar serviços em execução na nuvem do que monitorar serviços locais. Por exemplo, você não possui o controle total do ambiente de hospedagem e os serviços geralmente dependem de outros serviços fornecidos por fornecedores de plataformas e outros.

Existem muitos fatores que afetam aplicativos hospedados na nuvem, como a latência da rede, o desempenho e a disponibilidade dos sistemas de armazenamento e computação subjacente e a largura de banda da rede entre eles. O serviço pode falhar total ou parcialmente devido a qualquer desses fatores. Portanto, você deve verificar em intervalos regulares se o serviço está executando corretamente para garantir o nível de disponibilidade necessário, que pode ser parte do seu SLA (Contrato de Nível de Serviço).

## <a name="solution"></a>Solução

Implementar monitoramento de integridade, enviando solicitações para um ponto de extremidade no aplicativo. O aplicativo deve realizar as verificações necessárias e retornar uma indicação de seu status.

Uma verificação de monitoramento de integridade geralmente combina dois fatores:

- As verificações (se houver) realizadas pelo aplicativo ou serviço em resposta à solicitação para o ponto de extremidade de verificação de integridade.
- Análise dos resultados pela ferramenta ou estrutura que executa a verificação de integridade.

O código de resposta indica o status do aplicativo e, opcionalmente, quaisquer componentes ou serviços utilizados por ele. A verificação de tempo resposta ou latência é realizada pela estrutura ou ferramenta de monitoramento. A figura fornece uma visão geral do padrão.

![Visão geral do padrão](./_images/health-endpoint-monitoring-pattern.png)

Outras verificações que podem ser realizadas pelo código de monitoramento de integridade no aplicativo incluem:
- Verificação do armazenamento em nuvem ou um banco de dados para disponibilidade e tempo de resposta.
- Verificação de outros serviços ou recursos localizados no aplicativo, ou situados em outros locais, mas utilizados pelo aplicativo.

Existem serviços e ferramentas que monitoram aplicativos Web, enviando uma solicitação para um conjunto de pontos de extremidade configurável e avaliando os resultados em relação a um conjunto de regras configuráveis. É relativamente fácil criar um ponto de extremidade de serviço cujo único propósito é realizar alguns testes funcionais no sistema.

As verificações típicas que podem ser realizadas pelas ferramentas de monitoramento incluem:

- Validando o código de resposta. Por exemplo, uma resposta HTTP de 200 (OK) indica que o aplicativo respondeu sem erro. O sistema de monitoramento também pode verificar se há outros códigos de resposta para obter resultados mais abrangentes.
- Verificação do conteúdo da resposta para detectar erros, mesmo quando um código de status de 200 (OK) é retornado. Isso pode detectar erros que afetam apenas uma seção da página da Web retornada ou resposta do serviço. Por exemplo, verificar o título de uma página ou procurar uma frase específica que indique que a página correta foi retornada.
- Medindo o tempo de resposta, que indica uma combinação da latência da rede e o tempo que o aplicativo demorou para executar a solicitação. Um valor crescente pode indicar um problema emergente com o aplicativo ou a rede.
- Verificação dos recursos ou serviços localizados fora do aplicativo, como uma rede de distribuição de conteúdo utilizada pelo aplicativo para distribuir conteúdo de caches globais.
- Verificação da expiração de certificados SSL.
- Medindo o tempo de resposta de uma pesquisa de DNS para a URL do aplicativo para medir a latência de DNS e falhas de DNS.
- Validação da URL retornada pela pesquisa de DNS para garantir entradas corretas. Isso pode ajudar a evitar o redirecionamento de solicitação mal-intencionada através de um ataque bem-sucedido no servidor DNS.

Também é útil, sempre que possível, executar essas verificações em diferentes locais ou locais hospedados para medir e comparar os tempos de resposta. O ideal é você monitorar aplicativos de locais próximos aos clientes para obter uma exibição precisa do desempenho de cada local. Além de fornecer um mecanismo de verificação mais robusto, os resultados podem ajudá-lo a decidir o local de implantação para o aplicativo&mdash;e se deverá implantá-lo em mais de um datacenter.

Os testes também devem ser executados em todas as instâncias de serviço que os clientes utilizam para garantir que o aplicativo esteja funcionando corretamente para todos os clientes. Por exemplo, se o armazenamento do cliente se espalhar por mais de uma conta de armazenamento, o processo de monitoramento deverá verificar tudo isso.

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esse padrão:

Como validar a resposta. Por exemplo, somente um código de status de 200 (OK) único é suficiente para verificar se o aplicativo está funcionando corretamente? Embora isso forneça a medida mais básica de disponibilidade do aplicativo, e é a implementação mínima desse padrão, fornece pouca informação sobre as operações, tendências e possíveis problemas futuros no aplicativo.

   >  Verifique se o aplicativo retorna um 200 (OK) corretamente somente quando o recurso de destino é encontrado e processado. Em alguns cenários, como quando ao utilizar uma página mestre para hospedar a página da Web de destino, o servidor envia um código de status de 200 (OK) ao invés de um código 404 (Não Encontrado), mesmo quando a página de conteúdo de destino não foi encontrada.

O número de pontos de extremidade para expor um aplicativo. Uma abordagem é expor ao menos um ponto de extremidade para os principais serviços que o aplicativo utiliza e um outro para os serviços de menor prioridade, permitindo que diferentes níveis de importância sejam atribuídos a cada resultado de monitoramento. Considere também expor mais pontos de extremidade, como um para cada serviço principal, para granularidade de monitoramento adicional. Por exemplo, uma verificação de integridade pode verificar o banco de dados, o armazenamento e um serviço de geocodificação externo que um aplicativo utiliza, cada um exigindo um nível diferente de tempo de atividade e tempo de resposta. O aplicativo ainda poderá ser íntegro se o serviço de geocodificação, ou alguma outra tarefa em segundo plano, não estiver disponível por alguns minutos.

Seja para usar o mesmo ponto de extremidade para o monitoramento, como é utilizado para acesso geral, mas para um caminho específico projetado para verificações de integridade, por exemplo, /HealthCheck/{GUID}/ no ponto de extremidade de acesso geral. Isso permite que alguns testes funcionais no aplicativo sejam executados pelas ferramentas de monitoramento, como adicionar um novo registro de usuário, conectar e colocar uma ordem de teste, além de verificar se o ponto de extremidade de acesso geral está disponível.

O tipo de informação a ser coletada no serviço em resposta às solicitações de monitoramento e como retornar essas informações. A maioria das ferramentas e estruturas existentes pesquisa apenas o código de status HTTP que o ponto de extremidade retorna. Para retornar e validar informações adicionais, talvez seja necessário criar um serviço ou utilitário de monitoramento personalizado.

Quanta informação coletar. Executar um processamento excessivo durante a verificação pode sobrecarregar o aplicativo e afetar outros usuários. O tempo que demora pode exceder o tempo limite do sistema de monitoramento para que o aplicativo não esteja disponível. A maioria dos aplicativos inclui instrumentação, como manipuladores de erro e contadores de desempenho que registram desempenho e informações de erro detalhadas, isso pode ser suficiente em vez de retornar informações adicionais de uma verificação de integridade.

Armazenando em cache o status do ponto de extremidade. Pode custar caro executar a verificação de integridade com muita frequência. Se o estado de integridade for relatado através de um painel, por exemplo, você não deseja que cada solicitação do painel apresente uma verificação de integridade. Em vez disso, verifique periodicamente a integridade do sistema e armazene em cache o status. Exponha um ponto de extremidade que retorna o status em cache.

Como configurar a segurança para os pontos de extremidade de monitoramento para protegê-los do acesso público, o que pode expor o aplicativo a ataques mal-intencionados, arriscar a exposição de informações confidenciais ou atrair ataques de DoS (negação de serviço). Normalmente, isso deve ser feito na configuração do aplicativo para que ele possa ser atualizado facilmente sem reiniciar o aplicativo. Considere usar uma ou mais das seguintes técnicas:

- Proteger o ponto de extremidade exigindo autenticação. É possível fazer isso, utilizando uma chave de segurança de autenticação no cabeçalho da solicitação ou passando as credenciais com a solicitação, desde que o serviço de monitoramento ou a ferramenta forneça suporte à autenticação.

  - Utilizar um ponto de extremidade oculto ou obscuro. Por exemplo, exponha o ponto de extremidade em um endereço IP diferente ao utilizado pela URL do aplicativo padrão, configure o ponto de extremidade em uma porta HTTP não padronizada e/ou utilize um caminho complexo para a página de teste. Normalmente, você pode especificar portas e endereços de ponto de extremidade adicionais na configuração do aplicativo e adicionar entradas para esses pontos de extremidade ao servidor DNS, se necessário para evitar ter que especificar o endereço IP diretamente.

  - Expor um método em um ponto de extremidade que aceite um parâmetro, como um valor de chave ou um valor de modo de operação. Dependendo do valor fornecido para esse parâmetro, quando uma solicitação é recebida, o código pode executar um teste específico ou conjunto de testes, ou retornar um erro 404 (Não Encontrado) se o valor do parâmetro não for reconhecido. Os valores de parâmetros reconhecidos podem ser configurados na configuração do aplicativo.

     >  Os ataques de negação de serviço provavelmente terão menos impacto em um ponto de extremidade separado que executa testes funcionais básicos sem comprometer a operação do aplicativo. O ideal é evitar usar um teste que possa expor informações confidenciais. Se for necessário retornar informações que possam ser úteis a um invasor, considere como irá proteger o ponto de extremidade e os dados contra acesso não autorizado. Neste caso, apenas confiar na obscuridade não é suficiente. Também será necessário considerar usar uma conexão HTTPS e criptografar quaisquer dados confidenciais, embora isso aumente a carga no servidor.

- Como acessar um ponto de extremidade protegido utilizando a autenticação. Nem todas as ferramentas e estruturas podem ser configuradas para incluir credenciais com a solicitação de verificação de integridade. Por exemplo, os recursos de verificação de integridade internos do Microsoft Azure não podem fornecer credenciais de autenticação. Algumas alternativas de terceiros são [Pingdom](https://www.pingdom.com/), [Panopta](http://www.panopta.com/), [NewRelic](https://newrelic.com/) e [Statuscake](https://www.statuscake.com/).

- Como garantir que o agente de monitoramento esteja sendo executado corretamente. Uma abordagem é expor um ponto de extremidade que simplesmente retorne um valor a partir da configuração do aplicativo ou um valor aleatório que pode ser utilizado para testar o agente.

   >  Certifique-se também de que o sistema de monitoramento realize verificações em si, como uma autoteste e um teste interno, para evitar que ele emita resultados falsos positivos.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Esse padrão é útil para:
- Monitorar sites e aplicativos Web para verificar a disponibilidade.
- Monitorar sites e aplicativos Web para verificar a operação correta.
- Monitorar serviços compartilhados ou camada intermediária para detectar e isolar uma falha que poderia interromper outros aplicativos.
- Complementar a instrumentação existente no aplicativo, como contadores de desempenho e manipuladores de erro. A verificação de integridade não substitui o requisito para registro em log e auditoria no aplicativo. A instrumentação pode fornecer informações valiosas para uma estrutura existente que monitora contadores e logs de erros para detectar falhas ou outros problemas. No entanto, ele não pode fornecer informações se o aplicativo não estiver disponível.

## <a name="example"></a>Exemplo

Os seguintes exemplos de código, tirados da classe `HealthCheckController` (uma amostra que demonstra esse padrão está disponível no [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring)), demonstra expor um ponto de extremidade para executar um intervalo de verificações de integridade.

O método `CoreServices` mostrado abaixo em C#, realiza uma série de verificações nos serviços utilizados no aplicativo. Se todos os testes forem executados sem erro, o método retornará um código de status de 200 (OK). Se algum dos testes aumentar uma exceção, o método retornará um código de status de 500 (Erro Interno). O método poderá, opcionalmente, retornar informações adicionais quando ocorrer um erro, se a ferramenta de monitoramento ou a estrutura for capaz de usá-la.

```csharp
public ActionResult CoreServices()
{
  try
  {
    // Run a simple check to ensure the database is available.
    DataStore.Instance.CoreHealthCheck();

    // Run a simple check on our external service.
    MyExternalService.Instance.CoreHealthCheck();
  }
  catch (Exception ex)
  {
    Trace.TraceError("Exception in basic health check: {0}", ex.Message);

    // This can optionally return different status codes based on the exception.
    // Optionally it could return more details about the exception.
    // The additional information could be used by administrators who access the
    // endpoint with a browser, or using a ping utility that can display the
    // additional information.
    return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
  }
  return new HttpStatusCodeResult((int)HttpStatusCode.OK);
}
```
O método `ObscurePath` mostra como você pode ler um caminho da configuração do aplicativo e usá-lo como ponto de extremidade para testes. Este exemplo, em C#, também mostra como você pode aceitar uma ID como um parâmetro e utilizá-la para verificar solicitações válidas.

```csharp
public ActionResult ObscurePath(string id)
{
  // The id could be used as a simple way to obscure or hide the endpoint.
  // The id to match could be retrieved from configuration and, if matched,
  // perform a specific set of tests and return the result. If not matched it
  // could return a 404 (Not Found) status.

  // The obscure path can be set through configuration to hide the endpoint.
  var hiddenPathKey = CloudConfigurationManager.GetSetting("Test.ObscurePath");

  // If the value passed does not match that in configuration, return 404 (Not Found).
  if (!string.Equals(id, hiddenPathKey))
  {
    return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
  }

  // Else continue and run the tests...
  // Return results from the core services test.
  return this.CoreServices();
}
```

O método `TestResponseFromConfig` mostra como você pode expor um ponto de extremidade que executa uma verificação para um valor de configuração especificado.

```csharp
public ActionResult TestResponseFromConfig()
{
  // Health check that returns a response code set in configuration for testing.
  var returnStatusCodeSetting = CloudConfigurationManager.GetSetting(
                                                          "Test.ReturnStatusCode");

  int returnStatusCode;

  if (!int.TryParse(returnStatusCodeSetting, out returnStatusCode))
  {
    returnStatusCode = (int)HttpStatusCode.OK;
  }

  return new HttpStatusCodeResult(returnStatusCode);
}
```
## <a name="monitoring-endpoints-in-azure-hosted-applications"></a>Monitorar pontos de extremidade em aplicativos hospedados no Azure

Algumas opções para monitorar pontos de extremidades em aplicativos do Azure são:

- Utilizar os recursos de monitoramento internos do Azure.

- Utilizar uma estrutura ou serviço de terceiros como o Microsoft System Center Operations Manager.

- Criar um serviço ou utilitário personalizado que seja executado de maneira autônoma ou em um servidor hospedado.

   >  Embora o Azure forneça um conjunto de opções de monitoramento razoavelmente abrangente, é possível utilizar serviços e ferramentas adicionais para fornecer informações adicionais. Os Serviços de Gerenciamento do Azure fornecem um mecanismo de monitoramento interno para regras de alerta. A seção de alertas da página de serviços de gerenciamento no Portal do Azure permite que você configure até dez regras de alerta por assinatura para seus serviços. Essas regras especificam uma condição e um limite para um serviço, como carga de CPU, ou o número de solicitações ou erros por segundo, e o serviço pode enviar automaticamente notificações por email para os endereços definidos em cada regra.

As condições que você pode monitorar variam de acordo com o mecanismo de hospedagem escolhido para seu aplicativo (como Sites, Serviços de Nuvem, Máquinas Virtuais ou Serviços Móveis), mas todos eles incluem a capacidade de criar uma regra de alerta que utilize um ponto de extremidade da web que você especifica nas configurações do seu serviço. Este ponto de extremidade deve responder em tempo hábil para que o sistema de alerta possa detectar que o aplicativo está executando corretamente.

>  Leia mais informações sobre [criar notificações de alerta ][portal-alerts].

Se você hospedar seu aplicativo nas funções de trabalho e web nos Serviços de Nuvem do Azure ou Máquinas Virtuais, será possível tirar proveito de um dos serviços internos no Azure, chamado Gerenciador de Tráfego. O Gerenciador de Tráfego é um serviço de balanceamento de carga e roteamento que pode distribuir solicitações para instâncias específicas do seu aplicativo hospedado nos Serviços de Nuvem baseado em uma variedade de regras e configurações.

Além das solicitações de roteamento, o Gerenciador de Tráfego exibe uma URL, porta e caminho relativo especificados regularmente para determinar quais instâncias do aplicativo definido em suas regras estão ativas e estão respondendo as solicitações. Se detectar um código de status 200 (OK), ele marcará o aplicativo como disponível. Qualquer outro código de status faz com que o Gerenciador de Tráfego marque o aplicativo como offline. É possível exibir o status no console do Gerenciador de Tráfego e configurar a regra para reencaminhar solicitações para outras instâncias do aplicativo que estão respondendo.

No entanto, o Gerenciador de Tráfego aguardará apenas dez segundos para receber uma resposta da URL de monitoramento. Portanto, você deverá garantir que seu código de verificação de integridade execute nesse tempo, permitindo a latência da rede para viagem de ida e volta do Gerenciador de Tráfego para o seu aplicativo e novamente.

>  Leia mais informações sobre o uso do [Gerenciador de Tráfego para monitorar seus aplicativos](https://azure.microsoft.com/documentation/services/traffic-manager/). O Gerenciador de Tráfego também é discutido nas [Diretrizes de Implantação de Datacenter Múltiplo](https://msdn.microsoft.com/library/dn589779.aspx).

## <a name="related-guidance"></a>Diretrizes relacionadas

As diretrizes a seguir podem ser úteis ao implementar esse padrão:
- [Diretrizes sobre Instrumentação e Telemetria](https://msdn.microsoft.com/library/dn589775.aspx). A verificação da integridade dos serviços e componentes geralmente é feita por investigação, mas também é útil ter informações no local para monitorar o desempenho do aplicativo e detectar eventos que ocorrem em tempo de execução. Esses dados podem ser transmitidos de volta às ferramentas de monitoramento como informações adicionais para o monitoramento de integridade. As Diretrizes de Telemetria e Instrumentação obtêm informações de diagnóstico remoto coletadas por instrumentação em aplicativos.
- [Receber notificações de alerta][portal-alerts].
- Esse padrão inclui um [aplicativo de exemplo](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring) para fazer o download.

[portal-alerts]: https://azure.microsoft.com/documentation/articles/insights-receive-alert-notifications/
