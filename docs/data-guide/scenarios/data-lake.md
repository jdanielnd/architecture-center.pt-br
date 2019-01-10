# <a name="data-lakes"></a>Data lakes

Um data lake é um repositório de armazenamento que contém uma grande quantidade de dados em seu formato nativo e bruto. Os repositórios Data lake são otimizados para dimensionamento para terabytes e petabytes de dados. Normalmente, os dados vêm de várias fontes heterogêneas e podem ser estruturados, semi-estruturados ou não estruturados. A ideia com um data lake é armazenar tudo em seu estado original, não transformado. Essa abordagem é diferente de um [data warehouse](../relational-data/data-warehousing.md) tradicional, que transforma e processa os dados no momento da ingestão.

Vantagens de um data lake:

- Os dados nunca são descartados porque são armazenados no formato bruto. Isso é especialmente útil em um ambiente de Big Data, quando você pode não saber com antecedência quais informações estão disponíveis nos dados.
- Os usuários podem explorar os dados e criar suas próprias consultas.
- Pode ser mais rápido do que as ferramentas tradicionais de ETL.
- Mais flexível do que um data warehouse porque ele pode armazenar dados não estruturados e semi-estruturados.

Uma solução de data lake completa consiste no processamento e armazenamento. O armazenamento do data lake é criado para tolerância a falhas, escalabilidade infinita e ingestão de alta taxa de transferência de dados com vários formatos e tamanhos. O processamento do data lake envolve um ou mais mecanismos de processamento criados com essas metas em mente e pode operar em dados armazenados em um data lake em escala.

## <a name="when-to-use-a-data-lake"></a>Quando usar um data lake

O uso típico de um data lake inclui: [exploração de dados](./interactive-data-exploration.md), análise de dados e aprendizado de máquina.

Um data lake também pode agir como a fonte de dados para um data warehouse. Com essa abordagem, os dados brutos são incluídos no data lake e, em seguida, são transformados em um formato de consulta estruturado. Normalmente, essa transformação usa um pipeline de [ELT](../relational-data/etl.md#extract-load-and-transform-elt) (extrair-carregar-transformar), onde os dados são incluídos e transformados no local. A fonte de dados que já é relacional pode ir diretamente para o data warehouse, usando um processo de ETL, ignorando o data lake.

Os data lake stores costumam ser usados em cenários de streaming de eventos ou de IoT, porque eles persistir grandes quantidades de dados relacionais e não relacionais sem transformação nem definição de esquema. Eles foram criados para lidar com grandes volumes de pequenas gravações em baixa latência e são otimizados para alta taxa de transferência.

## <a name="challenges"></a>Desafios

- A falta de um esquema ou metadados descritivos pode tornar os dados difíceis de consumir ou consultar.
- A falta de consistência semântica nos dados pode transformar a execução da análise de dados em um desafio, a menos que os usuários sejam altamente especializados em análise de dados.
- Pode ser difícil garantir a qualidade dos dados sendo transferidos para o data lake.
- Sem a governança adequada, questões de privacidade e controle de acesso podem ser um problema. Quais informações estão indo para o data lake, quem pode acessar os dados e para qual uso?
- Um data lake pode não ser a melhor maneira de integrar os dados que já são relacionais.
- Por si só, um data lake não fornece exibições integradas ou holísticas para toda a organização.
- Um data lake pode se tornar uma lixeira para os dados que nunca são analisados ou dos quais não são extraídas informações.

## <a name="relevant-azure-services"></a>Serviços do Azure relevantes

- O [Data Lake Store](/azure/data-lake-store/) é um repositório de hiperescala compatível com o Hadoop.
- O [Data Lake Analytics](/azure/data-lake-analytics/) é um serviço de trabalho de análise sob demanda que simplifica a análise de Big Data.
