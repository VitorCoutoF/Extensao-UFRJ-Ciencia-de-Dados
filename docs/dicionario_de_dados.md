# Dicionário de dados

Descrição das tabelas usadas no projeto, das colunas que analisamos e dos problemas que já encontramos. Atualize este arquivo sempre que descobrir algo novo sobre os dados.

---

## Chamados do 1746
**Tabela:** `datario.adm_central_atendimento_1746.chamado`
**Tipo:** eventos — cada linha é um chamado aberto por um cidadão
**Período:** de 01/06/2010 até a data da última atualização (20/09/2026 na nossa extração)
**Recorte usado:** `tipo = 'Manejo Arbóreo'`, de 2021 em diante

| Coluna | O que é |
|---|---|
| `id_chamado` | identificador do chamado (deveria ser único) |
| `data_inicio` | data e hora de abertura |
| `data_fim` | data e hora de encerramento (vazia se o chamado não foi encerrado) |
| `id_bairro` | código do bairro; liga com a tabela de bairros |
| `id_logradouro` | código do logradouro; liga com a tabela de logradouros |
| `categoria` | natureza do contato: Reclamação, Serviço, Elogio, Crítica, Sugestão, Informações, Denúncia |
| `tipo` | assunto geral (usamos "Manejo Arbóreo") |
| `subtipo` | serviço específico (poda, remoção, árvore caída...) |
| `situacao` | Encerrado ou Não Encerrado |
| `tipo_situacao` | desfecho (ex.: Atendido, Não atendido) |
| `dentro_prazo` | situação do prazo (A vencer, Em vencimento, Vencido, Não calculado, Pendente de programação) |
| `nome_unidade_organizacional` | órgão responsável pelo atendimento |

**Problemas conhecidos**
- O último mês da base está sempre incompleto.
- 72.626 chamados de manejo arbóreo (histórico completo) estão sem bairro.
- Há indício de encerramentos em lote: na amostra, chamados da SEOP de 2011 foram todos encerrados em 05/09/2018 às 22:00.
- Os subtipos mudaram de nome ao longo do tempo (ex.: "Árvore - Poda" e "Poda de árvore em logradouro").
- Nem toda categoria é pedido de serviço (Elogio, Crítica, Sugestão).

---

## Chuva (Alerta Rio)
**Tabela:** `datario.clima_pluviometro.taxa_precipitacao_alertario`
**Tipo:** série temporal — cada linha é uma medição de uma estação pluviométrica, a cada 15 minutos (96 por dia)
**Recorte usado:** de 2021 em diante, sem valores negativos

| Coluna | O que é |
|---|---|
| `id_estacao` | código da estação pluviométrica |
| `data_particao` | data da medição |
| `acumulado_chuva_15_min` | chuva nos últimos 15 minutos (mm) — **é a coluna usada para somar a chuva do dia** |
| `acumulado_chuva_1_h`, `_4_h`, `_24_h`, `_96_h` | chuva acumulada nas últimas 1, 4, 24 e 96 horas (mm) |

**Cuidados**
- Os acumulados de 1 h, 4 h, 24 h e 96 h são **janelas que se sobrepõem**: não somar nem tirar média deles como se fossem medições independentes.
- Existem valores negativos, provavelmente códigos de erro; são descartados.
- Algumas estações têm dias com medições faltando ou repetidas. Consideramos "dia bom" o que tem entre 90 e 96 medições.

---

## Chuva (INMET / BDMEP)

**Tabelas:** `basedosdados.br_inmet_bdmep.microdados` e `basedosdados.br_inmet_bdmep.estacao`

**Notebook:** `notebooks/05_eda_clima_v2.ipynb`

**Recorte:** estações com município cadastrado no estado do Rio de Janeiro (prefixo IBGE `33`), desde 01/01/2021. Referência inicial: 14/10/2026; último dia solicitado: 13/10/2026. A cobertura efetiva é calculada na execução.

| Coluna de origem | O que é |
|---|---|
| `ano`, `data`, `hora` | Partição anual, data e horário registrado em UTC |
| `id_estacao` | Identificador que relaciona medições e cadastro |
| `precipitacao_total` | Precipitação horária, em mm |
| `id_municipio`, `estacao`, `data_fundacao` | Município cadastrado, nome e data de fundação da estação |

**Campos derivados e critérios**

- `dia`: dia civil do horário registrado convertido para `America/Sao_Paulo`; pode diferir do dia meteorológico oficial.
- `chuva_observada_mm`: soma das horas válidas disponíveis na tabela diária. Na tabela mensal, soma a média diária dos dias disponíveis; pode ser parcial.
- `chuva_mm`: total diário por estação somente com 24 horas válidas. Em `chuva_rede_dia`, é a média das estações com dia completo; em `chuva_rede_mes`, é a soma dessas médias quando todos os dias do mês estão disponíveis.
- `horas_validas`, `dia_completo`, `dias_validos`, `mes_completo`: indicam quanto do período tem dados. Um dia sem medições não é um dia seco.
- `estacoes_validas`: número de estações usadas na média de cada dia. A composição da média pode mudar ao longo do período.

**Cuidados**

- Nulos, negativos e valores não finitos são excluídos das somas, sem preencher com zero. Extremos positivos permanecem para investigação.
- Duplicatas idênticas por estação e horário são consolidadas; valores divergentes invalidam aquela hora.
- Dias anteriores à fundação não entram no denominador esperado da cobertura. Fundação ausente usa o início solicitado.
- A média representa os pontos com dados disponíveis, sem ponderação por área.
- O notebook simplificado salva cinco tabelas: estações, qualidade das estações, chuva por estação e dia, média diária e chuva mensal. Artefatos da versão anterior podem continuar na pasta, mas não são atualizados por esta versão.
- Bruto em `data/raw/inmet_rj_horario_*.parquet`; tabelas em `data/processed/inmet_rj/`; gráficos nas saídas do notebook.

---

## Logradouros
**Tabela:** `datario.dados_mestres.logradouro`
**Tipo:** cadastro — cada linha é um **trecho** de rua (o pedaço entre dois cruzamentos), não a rua inteira
**Tamanho:** cerca de 132 mil trechos, 44 mil logradouros e 167 bairros

| Coluna | O que é |
|---|---|
| `id_trecho` | identificador do trecho (deveria ser único) |
| `id_logradouro` | identificador da rua; se repete em todos os trechos da mesma rua |
| `nome_completo` | nome da rua |
| `id_bairro`, `nome_bairro` | bairro onde está o trecho |
| `tipo_logradouro_extenso` | Rua, Avenida, Travessa... |
| `hierarquia` | importância da via (Local, Arterial...) |
| `velocidade_regulamentada` | velocidade máxima (vem como texto; 0 parece significar "não informado") |
| `inicio_/final_numero_porta_par/impar` | faixa de numeração das casas no trecho |
| `geometry`, `geometry_wkt` | desenho do trecho no mapa (as duas guardam a mesma informação) |

**Problemas conhecidos**
- Juntar os chamados direto com esta tabela pelo `id_logradouro` **multiplica os chamados** (um por trecho). Use `logradouros_unicos`.
- `data_ultima_edicao` está 100% vazia.
- A numeração das casas falta em cerca de 63% dos trechos.
- Há 7 `id_trecho` repetidos.

---

## Bairros
**Tabela:** `datario.dados_mestres.bairro`
**Tipo:** cadastro de apoio — cada linha é um bairro

| Coluna | O que é |
|---|---|
| `id_bairro` | código do bairro (chave que liga chamados e logradouros) |
| `nome` | nome do bairro |
