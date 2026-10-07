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
