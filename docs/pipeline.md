# Pipeline de Dados — Descricao Detalhada

## Metodologia

O pipeline segue as etapas obrigatorias de um pipeline de Big Data, implementadas sobre a plataforma **Databricks** com arquitetura **Medallion**. Nao ha execucao local — todo o processamento e realizado no ambiente distribuido do Databricks.

---

## Etapas do Pipeline

### 1. Fonte de Dados (Data Source)

**Dataset:** [Instacart Market Basket Analysis](https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis) — Kaggle

Dados estruturados em formato CSV, descrevendo o historico de compras de mais de 200 mil clientes em um supermercado online. Inclui informacoes de pedidos, produtos, corredores, departamentos e padroes de recompra.

Volumes dos dados:
- ~3,4 milhoes de pedidos
- ~32 milhoes de itens em pedidos
- ~50 mil produtos catalogados

---

### 2. Ingestao (Ingestion)

**Tipo:** Batch

**Notebook:** `SRC/ETL/0.RAW/ingest_Instacart_kaggle.ipynb`

O notebook realiza o download dos arquivos CSV do Kaggle via API e os armazena no volume do Databricks em `/Volumes/big_data/raw/data/instacart/`. Os arquivos sao mantidos em formato original (CSV), sem modificacoes, como camada de auditoria imutavel.

---

### 3. Armazenamento Bruto — BRONZE

**Notebook:** `SRC/ETL/1.BRONZE/update_bronze_tables.ipynb`

**Schema:** `big_data.bronze`

Os CSVs da camada RAW sao lidos via Spark e gravados como Delta Tables no schema Bronze. Nao sao aplicadas transformacoes de negocio nesta etapa — o objetivo e garantir a persistencia dos dados brutos em formato otimizado (Delta), com adicao de metadados de controle (`ingestion_timestamp`, `source_file`).

---

### 4. Transformacao (Transformation) — SILVER

**Notebook:** `SRC/ETL/2.SILVER/update_silver_tables.ipynb`

**Schema:** `big_data.silver`

Etapa central de qualidade e enriquecimento dos dados. As transformacoes incluem:

- **Type casting:** Conversao de tipos para os adequados (ex: IDs como inteiros, timestamps corretos)
- **Filtragem de nulos:** Remocao de registros invalidos
- **Feature engineering:**
  - `is_first_order` — identifica o primeiro pedido de cada cliente
  - `period_of_day` — classifica o horario do pedido em `morning`, `afternoon`, `evening` ou `night`
- **Enriquecimento:** Join de `products` com `aisles` e `departments` para gerar `products_enriched`
- **Unificacao:** Union de `order_products__prior` e `order_products__train` em uma unica tabela `order_products`

Metadado adicionado: `_silver_timestamp`

---

### 5. Carregamento e Destino — GOLD

**Notebook:** `SRC/ETL/3.GOLD/analysis_gold_tables.ipynb`

**Schema:** `big_data.gold`

Camada analitica. Os dados da Silver sao agregados e transformados em tabelas de negocio prontas para consumo por analistas e ferramentas de visualizacao.

Grupos de tabelas geradas:

**Sales Analytics**
- Distribuicao de vendas por periodo do dia e hora
- Taxa de recompra por departamento

**Customer Analytics**
- Segmentacao de clientes por frequencia de compra
- Distribuicao de intervalo entre pedidos

**Product Analytics**
- Performance individual de produtos (volume, recompra)
- Performance por departamento e corredor

Os resultados ficam disponiveis como tabelas Delta no schema `big_data.gold`, consumiveis diretamente via SQL no Databricks ou por notebooks de visualizacao.

---

## Checklist AV1

| Etapa | Status |
|---|---|
| Ingestao (RAW) | Finalizado |
| Armazenamento Bronze | Finalizado |
| Transformacao Silver | Finalizado |
| Camada Gold (Analitica) | Em progresso |

---

## Divisao de Tarefas

| Membro | Responsabilidade |
|---|---|
| Caio Hirata | Arquitetura Medallion, notebook Bronze |
| Camila Cirne | Transformacoes Silver, qualidade de dados |
| Diogo Correia | Ingestao RAW, configuracao Databricks |
| Flavio Muniz | Camada Gold, metricas de negocio |
| Pedro Coelho | Documentacao, diagrama de arquitetura |
| Virna Amaral | Analise exploratoria, visualizacoes |
