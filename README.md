# Árvores, chuva e o 1746 — Ciência de Dados para Cidades Inteligentes

Projeto de extensão que investiga a relação entre a **demanda por manejo de árvores**, registrada na Central 1746 da Prefeitura do Rio de Janeiro, e a **chuva medida pelas estações do Alerta Rio**. Os cadastros de bairros e logradouros ajudam a localizar os chamados e comparar sua distribuição na cidade.

A pergunta principal é: **quando chove mais, aumentam os chamados sobre árvores, e em quais lugares isso acontece?** A análise também procura entender quais serviços são solicitados, como os registros mudaram ao longo dos anos e quais problemas dos dados podem afetar as conclusões.

**Etapa atual:** análise exploratória de dados (EDA), com resultados salvos para chamados e chuva, carregamento do cadastro de logradouros e código preparado para os primeiros cruzamentos. A relação entre chuva e chamados ainda precisa ser validada.

## Dados utilizados

As consultas usam tabelas públicas do projeto `datario` no BigQuery.

| Tabela | O que cada linha representa | Papel na análise |
|---|---|---|
| `datario.adm_central_atendimento_1746.chamado` | Um chamado do 1746 | Datas, localização, serviço solicitado, situação e prazo |
| `datario.clima_pluviometro.taxa_precipitacao_alertario` | Uma medição de uma estação, com acumulados de chuva em diferentes intervalos | Construção de séries diárias e mensais de chuva |
| `datario.dados_mestres.logradouro` | Um trecho de rua | Identificação de logradouros e caracterização dos bairros |
| `datario.dados_mestres.bairro` | Um bairro | Associação entre códigos e nomes dos bairros |

O recorte principal dos chamados usa `tipo = 'Manejo Arbóreo'` e abertura **a partir de 2021**. Algumas consultas de diagnóstico, como a busca de identificadores duplicados e de chamados sem localização, usam o histórico completo desse tipo. A chuva também é agregada a partir de 2021, mas o período efetivamente disponível precisa ser conferido antes do cruzamento.

As colunas e observações de cada base estão no [dicionário de dados](docs/dicionario_de_dados.md). Os resultados numéricos deste README se referem às saídas salvas nos notebooks; uma nova consulta pode produzir valores diferentes.

## Estrutura da pasta

A organização segue a convenção mais usada em projetos de ciência de dados, inspirada no *Cookiecutter Data Science*: dados, notebooks, código e documentação ficam em pastas separadas, e os notebooks são numerados na ordem em que devem ser executados.

```text
Vitor Couto Filho/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── raw/          ← dados brutos, como vieram do BigQuery (ainda não usada)
│   └── processed/    ← resultados salvos pelos notebooks, em .parquet
├── docs/
│   └── dicionario_de_dados.md
├── notebooks/
│   ├── 00_autenticacao_bigquery.ipynb
│   ├── 01_eda_chamados_1746.ipynb
│   ├── 02_eda_clima.ipynb
│   ├── 03_eda_logradouros.ipynb
│   └── 04_cruzamentos.ipynb
└── src/
    ├── extracao.py
    └── tratamento.py
```

| Arquivo | Conteúdo atual |
|---|---|
| [notebooks/00_autenticacao_bigquery.ipynb](notebooks/00_autenticacao_bigquery.ipynb) | Instalação das bibliotecas, imports, definição do projeto e teste de conexão com `SELECT 1`. Os demais notebooks executam este arquivo com `%run` |
| [notebooks/01_eda_chamados_1746.ipynb](notebooks/01_eda_chamados_1746.ipynb) | EDA dos chamados de Manejo Arbóreo; salva os chamados desde 2021 e a contagem mensal |
| [notebooks/02_eda_clima.ipynb](notebooks/02_eda_clima.ipynb) | EDA da chuva do Alerta Rio; salva a chuva por estação e dia e a chuva mensal da cidade |
| [notebooks/03_eda_logradouros.ipynb](notebooks/03_eda_logradouros.ipynb) | EDA dos cadastros de logradouros e bairros; salva as colunas usadas nos cruzamentos |
| [notebooks/04_cruzamentos.ipynb](notebooks/04_cruzamentos.ipynb) | Primeiros cruzamentos entre chamados, bairros, logradouros e chuva, a partir dos arquivos salvos |
| [docs/dicionario_de_dados.md](docs/dicionario_de_dados.md) | Descrição das tabelas, colunas e problemas identificados |
| [src/extracao.py](src/extracao.py) | Arquivo reservado para a extração de dados, ainda sem código |
| [src/tratamento.py](src/tratamento.py) | Arquivo reservado para o tratamento de dados, ainda sem código |
| [requirements.txt](requirements.txt) | Bibliotecas do projeto, ainda sem versões fixadas |

Ainda não há configuração por `.env` nem exportação automática dos gráficos. O conteúdo de `data/raw/` e `data/processed/` fica fora do Git (veja o `.gitignore`), porque os arquivos são recriados ao executar os notebooks.

## Como executar no VS Code

### 1. Preparar o ambiente Python

Abra a pasta do projeto no VS Code e tenha as extensões **Python** e **Jupyter** disponíveis. No terminal do Windows, dentro dessa pasta, crie um ambiente virtual e instale as dependências:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Os comandos usam diretamente o Python do ambiente virtual. Nos notebooks, selecione esse mesmo ambiente em **Selecionar Kernel**. O notebook de autenticação também executa `%pip install -r ../requirements.txt`, que instala os pacotes no ambiente do kernel selecionado; se a instalação solicitar reinicialização, reinicie o kernel antes de continuar.

### 2. Configurar e testar a autenticação

Abra [notebooks/00_autenticacao_bigquery.ipynb](notebooks/00_autenticacao_bigquery.ipynb). Na célula de configuração, preencha:

```python
BILLING_PROJECT_ID = "seu-projeto-google-cloud"
```

Use o **ID do projeto**, com uma conta Google autorizada a executar consultas nele. Este é o **único lugar** onde o ID precisa ser preenchido, porque os outros notebooks executam este arquivo no início. Se a variável ficar vazia, o notebook para com uma mensagem pedindo o preenchimento.

Execute as células em ordem. O teste de conexão é:

```python
teste_autenticacao = bd.read_sql(
    "SELECT 1 AS conexao_ok",
    billing_project_id=BILLING_PROJECT_ID,
)
display(teste_autenticacao)
```

Siga as instruções de login exibidas no primeiro acesso. O resultado esperado é uma coluna `conexao_ok` com valor `1`. Esse teste confirma a execução de uma consulta simples no projeto; ele não lê as tabelas da análise. A autenticação usa o fluxo do `basedosdados` e dispensa `google.colab.auth`.

### 3. Executar os notebooks de EDA

Execute os notebooks da pasta `notebooks/` **na ordem da numeração**, com o mesmo ambiente Python selecionado em todos:

1. `01_eda_chamados_1746.ipynb`
2. `02_eda_clima.ipynb`
3. `03_eda_logradouros.ipynb`
4. `04_cruzamentos.ipynb`

A primeira célula de código de cada um é:

```python
%run ./00_autenticacao_bigquery.ipynb
```

Ela executa o notebook de autenticação dentro da sessão atual e disponibiliza os imports, o `bd` e o `BILLING_PROJECT_ID`. O caminho `./` supõe que o diretório de trabalho do kernel seja a pasta `notebooks/`, que é o padrão do VS Code. Abrir dois notebooks com o mesmo ambiente Python, por si só, não compartilha suas variáveis; por isso cada notebook faz o seu `%run`.

Os notebooks 01, 02 e 03 terminam salvando seus principais resultados em `data/processed/`, no formato `.parquet`, que preserva os tipos das colunas, como as datas. O 04 não consulta de novo essas bases no BigQuery: ele lê esses arquivos e para com uma mensagem indicando qual notebook executar se algum estiver faltando.

Dentro de cada notebook, execute as seções na ordem em que aparecem, porque as células posteriores dependem de objetos criados antes. A saída de erro salva na amostra da chuva (seção 2.1) vem de uma versão anterior da consulta; a execução completa dos quatro notebooks desde um kernel limpo continua pendente de validação.

## O que já foi construído no EDA

### Chamados do 1746

A análise começa com uma amostra das 34 colunas e verificações de qualidade: ausência de data de abertura, identificadores repetidos e falta de bairro ou logradouro. Há consultas para localizar os dias com mais aberturas e encerramentos e comparar o volume anual de Manejo Arbóreo com o total do 1746.

No recorte a partir de 2021, foram construídos:

- Séries mensais de chamados e gráficos de sazonalidade, comparando os meses de cada ano.
- Distribuições por dia da semana e hora de abertura.
- Contagens de subtipos por ano e busca de assuntos relacionados a árvores em outros tipos do 1746.
- Percentuais de bairro e logradouro ausentes por ano.
- Cruzamentos de situação, prazo e ano de abertura.
- Cálculo de dias entre abertura e encerramento, medianas anuais e inspeção de horários de fechamento repetidos.

O último mês observado é retirado das séries mensais para evitar comparar um mês potencialmente incompleto com meses completos. Isso não torna o último ano completo: comparações anuais também precisam considerar o período coberto.

A função `agrupar_subtipo()` reúne nomes semelhantes em grupos: **Poda**, **Remoção**, **Poda e remoção**, **Árvore caída**, **Avaliação de risco**, **Destoca**, **Calçada danificada por raiz**, **Outros** e **Sem subtipo**. Também existe uma classificação proposta em fases: “Antes da queda ou remoção”, “Depois da queda ou remoção” e “Não se aplica”. Essas regras precisam de revisão com o grupo; a fase é uma interpretação do nome do serviço, não uma sequência de eventos comprovada para cada árvore.

### Chuva do Alerta Rio

O notebook consulta o tamanho da tabela, o número de estações, os extremos de data e a frequência de valores nulos, negativos, zero e positivos. Também investiga uma data muito distante no futuro.

A série diária é construída somando `acumulado_chuva_15_min` por **estação e dia**, com filtro para valores não negativos. Valores nulos também ficam fora dessa condição. Os acumulados de 1, 4, 24 e 96 horas possuem sobreposição e não são somados para formar o total diário.

O código conta as medições de cada estação em cada dia. Para intervalos de 15 minutos, a referência é **96 medições por dia**. A regra exploratória de `dia_bom` aceita de **90 a 96 medições**, inclusive. Ela admite dias parcialmente incompletos e não substitui uma verificação de duplicatas por horário.

A partir dessa regra, são calculados a proporção de dias aceitos por estação, a média diária entre as estações disponíveis, os dias mais chuvosos e a soma mensal dessa média diária. Há também uma estimativa anual por estação, calculada como média diária × 365, usando estações com pelo menos 365 dias aceitos. Essa estimativa não equivale ao total observado de um ano específico.

### Logradouros, bairros e cruzamentos

O cadastro de logradouros já foi carregado, excluindo as colunas de geometria. As etapas seguintes têm código para verificar nulos, trechos repetidos, quantidade de trechos por rua, ruas em mais de um bairro, categorias de via, velocidade e faixas de numeração.

Também foram preparadas células para:

- Criar `logradouros_unicos`, com uma linha por `id_logradouro`, evitando multiplicar chamados em um cruzamento com vários trechos da mesma rua.
- Carregar os bairros e conferir a correspondência dos seus identificadores com os logradouros.
- Acrescentar os nomes dos bairros aos chamados com `merge`.
- Comparar volume absoluto de chamados com chamados por 100 trechos, considerando bairros com pelo menos 100 trechos cadastrados.
- Verificar se os códigos de logradouro dos chamados existem no cadastro.
- Juntar chuva e chamados por mês, desenhar gráficos e calcular a correlação nos meses comuns às duas séries.

Essas verificações e cruzamentos finais ainda estão sem resultados salvos. A deduplicação proposta mantém o primeiro trecho de cada logradouro; em ruas que atravessam bairros, isso escolhe apenas um bairro e precisa ser revisto para uma análise espacial mais precisa.

## Resultados registrados até agora

As observações abaixo vêm das saídas salvas nos notebooks da pasta [notebooks/](notebooks/). São resultados exploratórios, com os filtros indicados, e não uma consulta atualizada automaticamente.

| Tema e recorte | Resultado salvo | Implicação para a análise |
|---|---|---|
| Manejo Arbóreo, histórico completo | **72.626 chamados com `id_bairro` e `id_logradouro` simultaneamente nulos** | Esses registros não podem ser localizados por essas duas chaves; a contagem não significa que todas as outras informações geográficas foram verificadas |
| Manejo Arbóreo, histórico completo | **0** registros sem `data_inicio`; consulta de duplicidade de `id_chamado` sem linhas | As verificações salvas não identificaram esses dois problemas no recorte consultado |
| Manejo Arbóreo, desde 2021 | Última abertura registrada em **20/09/2026, às 12:43:08** | Setembro de 2026 foi excluído da série mensal |
| Manejo Arbóreo, desde 2021 | **206.942** chamados classificados como encerrados e **41.409** como não encerrados | A situação registrada precisa ser confrontada com as datas de encerramento |
| Manejo Arbóreo, desde 2021 | **41.418** chamados sem `data_fim` | “Não encerrado” e “sem data de encerramento” não têm a mesma contagem; investigar a diferença |
| Chuva, tabela completa | **29.345.149** linhas, **33** estações e datas de **01/01/1997 a 03/03/2099** | O máximo de data indica uma anomalia a investigar |
| Chuva, consulta com corte em julho de 2026 | **1** linha retornada, com data **03/03/2099** | O registro deve ser inspecionado antes de definir seu tratamento |
| Chuva, acumulado de 15 minutos | **114.409** nulos, **0** negativos e máximo de **588,4 mm** | A saída atual não confirma a hipótese de valores negativos; o extremo também merece investigação |
| Chuva diária, desde 2021 e valores não negativos | **39.228** combinações de estação e dia; **3.691** com mais de 96 medições; **85,6%** com 90 a 96 | A frequência das medições afeta a comparabilidade dos totais diários |
| Chuva diária, após os filtros da consulta | Período salvo de **01/01/2021 a 03/06/2024**, sem linhas a partir de 2025 | A série utilizada não cobre todo o período dos chamados |
| Cadastro de logradouros carregado | **132.000 linhas e 22 colunas**, após excluir as geometrias | A consulta independente de contagem e as verificações do cadastro ainda precisam ser executadas |

Os gráficos e a tabela mensal de chamados sugerem maior volume no começo de vários anos, com exceções, como julho e agosto de 2026. Esse padrão orienta a investigação, mas a associação com chuva ainda depende do cruzamento e da avaliação da cobertura das duas séries.

As medianas salvas de tempo até o encerramento diminuem de **28,9 dias em 2021** para **5,8 dias em 2026**. Essa diferença, isoladamente, não comprova melhora do serviço: chamados ainda abertos não entram no cálculo da duração, os períodos têm tempos de acompanhamento diferentes e há indícios de encerramentos concentrados em determinadas datas.

## Decisões e limitações da análise

- **Localização:** medir a perda de registros antes de excluir chamados sem bairro ou logradouro. As colunas `latitude` e `longitude` aparecem na amostra da tabela, mas ainda não são carregadas na consulta principal do recorte desde 2021; sua qualidade precisa ser avaliada.
- **Datas da chuva:** a consulta diária tem limite inicial em 2021, mas ainda não contém um limite superior explícito. A saída agregada terminar em 2024 não significa que a data de 2099 tenha sido corrigida na origem.
- **Cobertura temporal:** comparar apenas o período comum entre chuva e chamados e medir as lacunas. O corte fixo em julho de 2026 usado no diagnóstico não representa permanentemente a definição de “data futura”.
- **Cobertura das estações:** a média diária da cidade usa as estações com `dia_bom` naquele dia; a composição pode variar. A soma mensal também pode ficar subestimada quando faltam dias.
- **Definição dos serviços:** a busca por palavras encontrou registros relacionados a árvores em outros tipos. A inclusão desses registros e a harmonização dos subtipos ainda precisam de critérios finais.
- **Escala espacial:** quantidade de trechos é um indicador exploratório do tamanho da rede de ruas. Ela não mede diretamente população, extensão viária, quantidade de árvores ou exposição ao risco.
- **Interpretação:** correlação mensal não demonstra causalidade e pode esconder respostas aos temporais nos dias seguintes. Ainda não há um resultado final de correlação salvo.
