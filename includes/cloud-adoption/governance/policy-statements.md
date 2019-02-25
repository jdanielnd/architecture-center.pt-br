<!-- TEMPLATE FILE - DO NOT ADD METADATA -->

## <a name="policy-statements"></a>Declarações de políticas

As seguintes instruções de política estabelecem os requisitos necessários para atenuar os riscos definidos. Essas políticas definem os requisitos funcionais para a governança MVP. Cada uma será representada na implementação da governança de MVP.

Aceleração de implantação:

- Todos os ativos devem ser agrupados e marcados de acordo com o agrupamento definido e estratégias de marcação.
- Todos os ativos devem usar um modelo de implantação aprovado.
- Depois de uma base de governança ser estabelecida para um provedor de nuvem, qualquer ferramenta de implantação deve ser compatível com as ferramentas definidas pela equipe de governança.

Linha de base de identidade:

- Todos os ativos implantados na nuvem devem ser controlados usando identidades e funções aprovadas pelas políticas de governança atual.
- Todos os grupos que têm privilégios elevados na infraestrutura do Active Directory Domain Services local devem ser mapeados para uma função RBAC aprovada.

Linha de base de segurança:

- Qualquer ativo implantado na nuvem deve ter uma classificação de dados aprovados.
- Nenhum ativo identificado com um nível protegido de dados pode ser implantado na nuvem, até requisitos de segurança e governança suficientes podem ser aprovados e implementados.
- Até que os requisitos de segurança de rede mínima possam ser validados e controlados, ambientes de nuvem são vistos como uma zona desmilitarizada e devem atender aos requisitos de conexão semelhante a outros data centers ou redes internas.

Gerenciamento de Custos:

- Para fins de acompanhamento, todos os ativos devem ser atribuídos a um proprietário de aplicativo dentro de uma das funções de negócios principais.
- Ao surgir preocupações com o custo, requisitos adicionais de controle serão estabelecidos com a equipe de finanças.

Consistência de Recursos:

- Como nenhuma carga de trabalho de missão crítica é implantada neste estágio, não há nenhum SLA, desempenho ou requisitos de BCDR a ser regido.
- Quando as cargas de trabalho de missão crítica são implantadas, os requisitos adicionais de controle serão estabelecidos com operações de TI.

## <a name="processes"></a>Processos

Nenhum orçamento foi alocado para a imposição dessas políticas de governança e monitoramento contínuo. Por causa disso, a equipe de governança de nuvem tem algumas maneiras ad-hoc para monitorar a aderência às declarações de política.

- **Educação**: A Equipe de governança de nuvem está investindo tempo para orientar as equipes de adoção de nuvem nos percursos de governança que dão suporte a essas políticas.
- **Revisões** da implantação: Antes de implantar qualquer ativo, a equipe de governança de nuvem examinará a jornada de governança com as equipes de adoção da nuvem.
