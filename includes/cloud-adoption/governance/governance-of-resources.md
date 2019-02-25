<!-- TEMPLATE FILE - DO NOT ADD METADATA -->

### <a name="governance-of-resources"></a>Governança de recursos

A imposição de governança entre assinaturas decorrerá do Azure Blueprints e dos ativos associados no blueprint.

1. Crie um Azure Blueprint nomeado "MVP de Governança".
    1. Imponha o uso de funções padrão do Azure.
    2. Imponha que os usuários somente possam autenticar em uma implementação do RBAC existente.
    3. Aplique esse blueprint a todas as assinaturas dentro do grupo de gerenciamento.
2. Crie um Azure Policy para aplicar ou impor o seguinte:
    1. A marcação de recursos deve exigir valores para Função Empresarial, Classificação de Dados, Criticidade, SLA, Ambiente e Aplicativo.
    2. O valor da marca do Aplicativo deve corresponder ao nome do grupo de recursos.
    3. Valide as atribuições de função para cada grupo de recursos e recurso.
3. Publique e aplique o Azure Blueprint "MVP de Governança" para cada grupo de gerenciamento.

Esses padrões permitem que os recursos sejam descobertos e rastreados e imponham o gerenciamento de função básico.

### <a name="demilitarized-zone-dmz"></a>DMZ (zona desmilitarizada)

É comum que assinaturas específicas exijam algum nível de acesso aos recursos locais. Esse pode ser o caso de cenários de migração ou cenários de desenvolvimento, quando alguns recursos dependentes ainda estão no datacenter local. Nesse caso, o MVP de governança adiciona as melhores práticas a seguir:

1. Estabeleça uma DMZ de Nuvem.
    1. A [arquitetura de referência de DMZ de Nuvem](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid) estabelece um padrão e um modelo de implantação para criar um Gateway de VPN no Azure.
    2. Valide se os requisitos adequados de conectividade e segurança de DMZ estão em vigor para um dispositivo de borda local no datacenter local.
    3. Valide se o dispositivo de borda local é compatível com os requisitos do Gateway de VPN do Azure.
    4. Depois que a conexão com a VPN local for verificada, capture o modelo do Resource Manager criado por essa arquitetura de referência.
2. Crie um segundo blueprint nomeado "DMZ".
    1. Adicione o modelo do Resource Manager do Gateway de VPN para o blueprint.
3. Aplique o blueprint de DMZ a qualquer assinatura que exija conectividade local. Esse blueprint deve ser aplicado adicionalmente ao modelo de MVP de governança.

Uma das maiores preocupações manifestadas pelas equipes de segurança de TI e governança tradicional é sobre o risco de adoção de nuvem em estágio inicial, comprometendo os ativos existentes. A abordagem acima permite que as equipes de adoção de nuvem criem e migrem soluções híbridas com risco reduzido para ativos locais. Na evolução mais recente, essa solução temporária será removida.

> [!NOTE]
> O descrito acima é um ponto de partida para criar rapidamente um MVP de governança de linha de base. Este é apenas o começo do percurso de governança. Mais evolução será necessária na medida em que a empresa continuar a adotar a nuvem e assumir mais riscos nas seguintes áreas:
>
> - Cargas de trabalho críticas
> - Dados protegidos
> - Gerenciamento de custo
> - Cenários multinuvem
>
>Além disso, os detalhes específicos deste MVP são baseados no exemplo de percurso de uma empresa fictícia, descrita nos artigos a seguir. É altamente recomendável familiarizar-se com os outros artigos desta série, antes de implementar essa melhor prática.
