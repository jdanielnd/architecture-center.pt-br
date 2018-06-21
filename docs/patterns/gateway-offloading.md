---
title: Padrão de descarregamento de gateway
description: Descarregue a funcionalidade de serviço especializado ou compartilhado para um proxy do gateway.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6b3e4541aae77349ca91c18c788ddb508912361d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540002"
---
# <a name="gateway-offloading-pattern"></a>Padrão de descarregamento de gateway

Descarregue a funcionalidade de serviço especializado ou compartilhado para um proxy do gateway. Esse padrão pode simplificar o desenvolvimento de aplicativo movendo a funcionalidade de serviços compartilhados, como o uso de certificados SSL, de outras partes do aplicativo para o gateway.

## <a name="context-and-problem"></a>Contexto e problema

Alguns recursos são comumente usados em vários serviços e esses recursos exigem configuração, gerenciamento e manutenção. Um serviço compartilhado ou especializado que é distribuído com cada implantação de aplicativo aumenta a sobrecarga administrativa e a probabilidade de erro de implantação. Todas as atualizações para um recurso compartilhado devem ser implantadas em todos os serviços que compartilham esse recurso.

Tratar corretamente os problemas de segurança (validação de token, criptografia e gerenciamento de certificados SSL) e outras tarefas complexas pode requerer que os membros da equipe tenham habilidades altamente especializadas. Por exemplo, um certificado necessário para um aplicativo deve ser configurado e implantado em todas as instâncias do aplicativo. Com cada implantação nova, o certificado deve ser gerenciado para se garantir que ele não expire. Qualquer certificado comum que esteja prestes a expirar deve ser atualizado, testado e verificado em todas as implantações de aplicativo.

Outros serviços comuns, como autenticação, autorização, registro em log, monitoramento ou [limitação](./throttling.md) podem ser difíceis de serem implementados e gerenciado em uma grande quantidade de implantações. Talvez seja melhor consolidar esse tipo de funcionalidade para reduzir a sobrecarga e a chance de erros.

## <a name="solution"></a>Solução

Descarregue alguns recursos em um gateway de API, particularmente preocupações abrangentes como gerenciamento de certificados, autenticação, término de SSL, monitoramento, conversão de protocolo ou limitação. 

O diagrama a seguir mostra um gateway de API que termina as conexões SSL de entrada. Ele solicita dados em nome do solicitante original de qualquer upstream de servidor HTTP do gateway de API.

 ![](./_images/gateway-offload.png)
 
Os benefícios desse padrão incluem:

- Simplificar o desenvolvimento de serviços removendo a necessidade de distribuir e manter recursos de suporte, como certificados de servidor Web e a configuração de sites seguros. Uma configuração mais simples resulta em gerenciamento e escalabilidade mais fáceis e simplifica as atualizações de serviço.

- Permitir que equipes dedicadas implementem recursos que exigem competências especializadas, como a segurança. Isso permite que sua equipe principal foque a funcionalidade do aplicativo, deixando essas preocupações especializadas, mas abrangentes, para os especialistas relevantes.

- Fornecer alguma consistência para registro em log e monitoramento de solicitação e de reposta. Mesmo que um serviço não esteja instrumentado corretamente, o gateway pode ser configurado para garantir um nível mínimo de monitoramento e registro em log.

## <a name="issues-and-considerations"></a>Problemas e considerações

- Assegure que o gateway de API seja altamente disponível e resistente a falhas. Evite pontos únicos de falha executando várias instâncias do seu gateway de API. 
- Assegure que o gateway seja projetado para os requisitos de capacidade e dimensionamento de seu aplicativo e pontos de extremidade. Verifique se o gateway não se torna um gargalo do aplicativo e se é suficientemente escalonável.
- Descarregue somente recursos que sejam usados pelo aplicativo inteiro, como a segurança ou transferência de dados.
- A lógica de negócios nunca deve ser descarregada para o gateway de API. 
- Se precisar controlar transações, leve em consideração gerar IDs de correlação para fins de registro de log.

## <a name="when-to-use-this-pattern"></a>Quando usar esse padrão

Use esse padrão quando:

- Uma implantação de aplicativo tem uma preocupação compartilhada, como certificados SSL ou criptografia.
- Em um recurso que seja comum entre implantações de aplicativos que podem ter requisitos de recursos diferentes, como recursos de memória, capacidade de armazenamento ou conexões de rede.
- Você deseja mover a responsabilidade por problemas, como segurança de rede, limitação ou outras preocupações de limite de rede, para uma equipe mais especializada.

Esse padrão pode não ser adequado caso ele introduza um acoplamento entre serviços.

## <a name="example"></a>Exemplo

Ao usar o Nginx como o dispositivo de descarregamento de SSL, a configuração a seguir encerra uma conexão SSL de entrada e distribui a conexão para um dos três servidores HTTP de upstream.

```
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

## <a name="related-guidance"></a>Diretrizes relacionadas

- [Padrão de back-ends para front-ends](./backends-for-frontends.md)
- [Padrão de agregação de gateway](./gateway-aggregation.md)
- [Padrão de roteamento de gateway](./gateway-routing.md)

