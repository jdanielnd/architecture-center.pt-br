---
title: Convenções de nomenclatura para recursos do Azure
description: Convenções de nomenclatura para recursos do Azure. Como nomear máquinas virtuais, contas de armazenamento, redes, redes virtuais, sub-redes e outras entidades do Azure
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: a92b6a1a23b35e7379f586d477b6f7cc6ccfc7e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206371"
---
# <a name="naming-conventions"></a>Convenções de nomenclatura

[!INCLUDE [header](../_includes/header.md)]

Este artigo é um resumo das regras e restrições de nomenclatura para recursos do Azure e um conjunto de recomendações para convenções de nomenclatura.  Você pode usar essas recomendações como um ponto de partida para suas próprias convenções específicas às suas necessidades.

A escolha de um nome para qualquer recurso do Microsoft Azure é importante, pois:

* É difícil alterar um nome posteriormente.
* Os nomes devem atender aos requisitos de seu tipo de recurso específico.

Convenções de nomenclatura consistentes facilitam a localização dos recursos. Elas também podem indicar a função de um recurso em uma solução.

A chave para o sucesso com convenções de nomenclatura é estabelecê-las e segui-las em seus aplicativos e organizações.

## <a name="naming-subscriptions"></a>Nomeando assinaturas
Ao nomear as assinaturas do Azure, nomes detalhados facilitam a compreensão do contexto e da finalidade de cada assinatura.  Ao trabalhar em um ambiente com várias assinaturas, seguir uma convenção de nomenclatura compartilhada pode proporcionar mais clareza.

Este é um padrão recomendado para nomear assinaturas:

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* A empresa seria a mesma para cada assinatura. No entanto, algumas empresas podem ter empresas filho dentro da estrutura organizacional. Essas empresas podem ser gerenciadas por um grupo central de TI. Nesses casos, elas podem ser diferenciadas tendo o nome da empresa pai (*Contoso*) e o nome da empresa filho (*Northwind*).
* O departamento é um nome dentro da organização que contém um grupo de indivíduos. Esse item no namespace é opcional.
* A linha de produtos é um nome específico de um produto ou uma função executada de dentro do departamento. Geralmente isso opcional para serviços e aplicativos internos. Porém, é altamente recomendado para uso em serviços públicos que exigem separação e identificação fácil (como no caso da separação clara de registros de cobrança).
* O ambiente é o nome que descreve o ciclo de vida de implantação dos aplicativos ou serviços, como Desenvolvimento, Controle de Qualidade ou Produção.

| Empresa | Departamento | Linha de produto ou serviço | Ambiente | Nome completo |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Produção |Contoso SocialGaming AwesomeService Produção |
| Contoso |SocialGaming |AwesomeService |Desenvolvimento |Contoso SocialGaming AwesomeService Desenvolvimento |
| Contoso |IT |InternalApps |Produção |Contoso IT InternalApps Produção |
| Contoso |IT |InternalApps |Desenvolvimento |Contoso IT InternalApps Desenvolvimento |

Para obter mais informações sobre como organizar as assinaturas de grandes empresas, leia nossas [diretrizes de governança de assinatura prescritiva][scaffold].

## <a name="use-affixes-to-avoid-ambiguity"></a>Use afixos para evitar ambiguidade

Ao nomear recursos no Azure, recomendamos o uso de prefixos ou sufixos comuns para identificar o tipo e o contexto do recurso.  Embora todas as informações sobre o tipo, os metadados e o contexto estejam disponíveis de modo programático, a aplicação de afixos comuns simplifica a identificação visual.  Ao incorporar afixos à sua convenção de nomenclatura, é importante especificar claramente se o afixo fica no início do nome (prefixo) ou no final (sufixo).

Por exemplo, veja dois nomes possíveis para um serviço que hospeda um mecanismo de cálculo:

* SvcCalculationEngine (prefixo)
* CalculationEngineSvc (sufixo)

Os afixos podem se referir a diversos aspectos que descrevam os recursos em questão. A tabela a seguir mostra alguns exemplos normalmente usados.

| Aspecto | Exemplo | Observações |
| --- | --- | --- |
| Ambiente |dev, prod, QA |Identifica o ambiente do recurso |
| Local padrão |uw (Oeste dos EUA), ue (Leste dos EUA) |Identifica a região na qual o recurso foi implantado |
| Instância |01, 02 |Para recursos com mais de uma instância nomeada (servidores Web etc.). |
| Produto ou serviço |propriedade serviço |Identifica o produto, o aplicativo ou o serviço que recebe suporte do recurso |
| Função |sql, web, messaging |Identifica a função do recurso associado |

Ao desenvolver uma convenção de nomenclatura específica para sua empresa ou seus projetos, é importante escolher um conjunto comum de afixos e a posição deles (sufixo ou prefixo).

## <a name="naming-rules-and-restrictions"></a>Regras e restrições de nomenclatura

Cada tipo de recurso ou serviço no Azure impõe um conjunto de restrições de nomes e escopo; qualquer convenção de nomenclatura ou padrão deve aderir às regras de nomeação e escopo de requisito.  Por exemplo, embora o nome de uma VM mapeie para um nome DNS (e, portanto, deve ser exclusivo em todo o Azure), o nome de uma rede virtual está no escopo do Grupo de Recursos no qual ele é criado.

Em geral, evite usar caracteres especiais (`-` ou `_`) como o primeiro ou último caractere em qualquer nome. Esses caracteres farão com que a maioria das regras de validação falhe.

### <a name="general"></a>Geral

| Entidade | Escopo | Comprimento | Capitalização | Caracteres válidos | Padrão sugerido | Exemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Grupo de recursos |Assinatura |1-90 |Não diferencia maiúsculas de minúsculas |Alfanumérico, sublinhado, parênteses, hífen e ponto (exceto no final) |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|Conjunto de disponibilidade |Grupo de recursos |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, sublinhado e hífen |`<service-short-name>-<context>-as` |`profx-sql-as` |
|Marca |Entidade associada |512 (nome), 256 (valor) |Não diferencia maiúsculas de minúsculas |Alfanumérico |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>Computação

| Entidade | Escopo | Comprimento | Capitalização | Caracteres válidos | Padrão sugerido | Exemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Máquina Virtual |Grupo de recursos |1-15 (Windows), 1-64 (Linux) |Não diferencia maiúsculas de minúsculas |Alfanumérico e hífen |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|Aplicativo de Funções | Global |1-60 |Não diferencia maiúsculas de minúsculas |Alfanumérico e hífen |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Máquinas virtuais no Azure têm dois nomes distintos: nome da máquina virtual e nome do host. Quando você cria uma máquina virtual no portal, o mesmo nome é usado para o nome do host e o nome do recurso de máquina virtual. As restrições acima são para o nome do host. O nome real do recurso pode ter até 64 caracteres.

### <a name="storage"></a>Armazenamento

| Entidade | Escopo | Comprimento | Capitalização | Caracteres válidos | Padrão sugerido | Exemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Nome da conta de armazenamento (dados) |Global |3-24 |Letras minúsculas |Alfanumérico |`<globally unique name><number>` (use uma função para calcular um guid exclusivo de nomeação de contas de armazenamento) |`profxdata001` |
|Nome da conta de armazenamento (discos) |Global |3-24 |Letras minúsculas |Alfanumérico |`<vm name without hyphens>st<number>` |`profxsql001st0` |
| Nome do contêiner |Conta de armazenamento |3-63 |Letras minúsculas |Alfanumérico e hífen |`<context>` |`logs` |
|Nome de blob | Contêiner |1-1024 |Diferencia maiúsculas de minúsculas |Caracteres de URL |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Nome da fila |Conta de armazenamento |3-63 |Letras minúsculas |Alfanumérico e hífen |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|Nome da tabela | Conta de armazenamento |3-63 |Não diferencia maiúsculas de minúsculas |Alfanumérico |`<service short name><context>` |`awesomeservicelogs` |
|Nome do arquivo | Conta de armazenamento |3-63 |Letras minúsculas | Alfanumérico |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | Global |3-24 |Letras minúsculas | Alfanumérico |`<name>dls` |`telemetrydls` |

### <a name="networking"></a>Rede

| Entidade | Escopo | Comprimento | Capitalização | Caracteres válidos | Padrão sugerido | Exemplo |
| --- | --- | --- | --- | --- | --- | --- |
|Rede virtual (VNet) |Grupo de recursos |2-64 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<service short name>-vnet` |`profx-vnet` |
|Sub-rede |Rede virtual pai |2-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<descriptive context>` |`web` |
|Interface de rede |Grupo de recursos |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<vmname>-nic<num>` |`profx-sql1-nic1` |
|Grupo de Segurança de Rede |Grupo de recursos |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|Regra de grupo de segurança de rede |Grupo de recursos |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<descriptive context>` |`sql-allow` |
|Endereço IP público |Grupo de recursos |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<vm or service name>-pip` |`profx-sql1-pip` |
|Load Balancer |Grupo de recursos |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<service or role>-lb` |`profx-lb` |
|Configuração de regras de balanceamento de carga |Load Balancer |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<descriptive context>` |`http` |
|Gateway de Aplicativo do Azure |Grupo de recursos |1-80 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen, sublinhado e ponto |`<service or role>-agw` |`profx-agw` |
|Perfil de Gerenciador de Tráfego |Grupo de recursos |1-63 |Não diferencia maiúsculas de minúsculas |Alfanumérico, hífen e ponto |`<descriptive context>` |`app1` |

## <a name="organize-resources-with-tags"></a>Organizar recursos com marcas

O Azure Resource Manager dá suporte à marcação de entidades com cadeias de caracteres de texto aleatórias com o objetivo de identificar o contexto e simplificar a automação.  Por exemplo, a marca `"sqlVersion: "sql2014ee"` pode identificar VMs em uma implantação que esteja executando o SQL Server 2014 Enterprise Edition para executar um script automatizado nelas.  As marcas devem ser usadas para ampliar e aprimorar o contexto junto com as convenções de nomenclatura escolhidas.

> [!TIP]
> Outra vantagem das marcas é que elas abrangem grupos de recursos, permitindo que você vincule e correlacione entidades em implantações diferentes.

Cada recurso ou grupo de recursos pode ter, no máximo, **15** marcas. O nome da marca é limitado a 512 caracteres e o valor da marca é limitado a 256 caracteres.

Para saber mais sobre marcação de recursos, veja [Uso de marcas para organizar os recursos do Azure](/azure/azure-resource-manager/resource-group-using-tags/).

Estes são alguns dos casos de uso de marcação mais comuns:

* **Cobrança**; agrupar recursos e associá-los a códigos de cobrança ou de estorno.
* **Identificação de contexto do serviço**; identificar grupos de recursos entre Grupos de recursos para operações comuns e agrupamento
* **Controle de acesso e contexto de segurança**; identificação de função administrativa com base no portfólio, sistema, serviço, aplicativo, instância etc.

> [!TIP]
> Marque no começo, marque com frequência.  É melhor ter uma esquema base de marcação aplicado e ajustar ao longo do tempo em vez de ter que fazer ajustes após o fato.

Veja exemplos de algumas abordagens comuns de marcação:

| Nome da marca | Chave | Exemplo | Comentário |
| --- | --- | --- | --- |
| Cobrar de/ID de estorno interno |billTo |`IT-Chargeback-1234` |Um código interno de E/S ou cobrança |
| Operador ou DRI (Pessoa diretamente responsável) |managedBy |`joe@contoso.com` |Alias ou endereço de email |
| Nome do projeto |projectName |`myproject` |Nome do projeto ou da linha de produto |
| Versão do projeto |projectVersion |`3.4` |Versão do projeto ou da linha de produto |
| Ambiente |Ambiente |`<Production, Staging, QA >` |Identificador de ambiente |
| Camada |Camada |`Front End, Back End, Data` |Identificação da camada ou da função/contexto |
| Perfil de dados |dataProfile |`Public, Confidential, Restricted, Internal` |Confidencialidade dos dados armazenados no recurso |

## <a name="tips-and-tricks"></a>Dicas e truques

Alguns tipos de recursos podem exigir cuidados adicionais em relação a nomenclatura e convenções.

### <a name="virtual-machines"></a>Máquinas virtuais

Especialmente em topologias maiores, a nomenclatura cuidadosa das máquinas virtuais simplifica a identificação da função e a finalidade de cada máquina e permite scripts mais previsíveis.

### <a name="storage-accounts-and-storage-entities"></a>Contas de armazenamento e entidades de armazenamento

Há dois casos de uso primário para contas de armazenamento: backup de discos para VMs e armazenamento de dados em blobs, em filas e em tabelas.  As contas de armazenamento usadas para discos de VM devem seguir a convenção de nomenclatura de associação com o nome da VM pai (e com a possível necessidade de várias contas de armazenamento para SKUs de VM de alto nível, também aplicam um sufixo numérico).

> [!TIP]
> Contas de armazenamento — quer seja para dados ou discos — devem seguir uma convenção de nomenclatura que permita que várias contas de armazenamento sejam aproveitadas (ou seja, sempre usar um sufixo numérico).

É possível configurar um nome de domínio personalizado para acessar dados de blob em sua Conta de Armazenamento do Microsoft Azure. O ponto de extremidade padrão para o serviço Blob é https://\<nome\>.blob.core.windows.net.

Porém, se você mapear um domínio personalizado (como www.contoso.com) para o ponto de extremidade do blob de sua conta de armazenamento, também poderá acessar dados do blob em sua conta de armazenamento usando esse domínio. Por exemplo, com um nome de domínio personalizado, `http://mystorage.blob.core.windows.net/mycontainer/myblob` pode ser acessada como `http://www.contoso.com/mycontainer/myblob`.

Para obter mais informações sobre como configurar esse recurso, confira [Configurar um nome de domínio personalizado para seu ponto de extremidade de Armazenamento de Blobs](/azure/storage/storage-custom-domain-name/).

Para obter mais informações sobre como nomear blobs, contêineres e tabelas, confira a lista a seguir:

* [Nomeando e referenciando contêineres, blobs e metadados](https://msdn.microsoft.com/library/dd135715.aspx)
* [Nomeação de filas e de metadados](https://msdn.microsoft.com/library/dd179349.aspx)
* [Tabelas de nomenclatura](https://msdn.microsoft.com/library/azure/dd179338.aspx)

Um nome de blob pode conter qualquer combinação de caracteres, mas os caracteres reservados de URL devem ser escapados corretamente. Evite nomes de blob que terminem com um ponto (.), com uma barra (/) ou com uma sequência ou uma combinação dos dois. Por convenção, a barra é o separador do diretório **virtual** . Não use uma barra invertida (\\) em um nome de blob. As APIs de cliente podem permitir o uso dela, mas não conseguem aplicar o hash corretamente, e as assinaturas não corresponderão.

Não é possível modificar o nome de uma conta de armazenamento ou um contêiner após sua criação. Se quiser usar um novo nome, você deverá excluí-lo e criar um novo.

> [!TIP]
> Recomendamos o estabelecimento de uma convenção de nomenclatura para todos os tipos e contas de armazenamento antes de embarcar no desenvolvimento de um novo serviço ou aplicativo.

<!-- links -->

[scaffold]: /azure/azure-resource-manager/resource-manager-subscription-governance
