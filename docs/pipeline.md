# Pipeline de Dados — Descrição Detalhada

## Metodologia

O pipeline segue as etapas obrigatórias de um pipeline de Big Data, implementadas sobre a plataforma **Databricks** com arquitetura **Medallion**. Não há execução local — todo o processamento é realizado no ambiente distribuído do Databricks.

---

## Etapas do Pipeline

### 1. Fonte de Dados (Data Source)

**Dataset:** [Instacart Market Basket Analysis](https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis) — Kaggle

Dados estruturados em formato CSV, descrevendo o histórico de compras de mais de 200 mil clientes em um supermercado online. Inclui informações de pedidos, produtos, corredores, departamentos e padrões de recompra.

Volumes dos dados:
- ~3,4 milhões de pedidos
- ~32 milhões de itens em pedidos
- ~50 mil produtos catalogados

---

### 2. Ingestão (Ingestion)

**Tipo:** Batch

**Notebook:** `SRC/ETL/0.RAW/ingest_Instacart_kaggle.ipynb`

O notebook realiza o download dos arquivos CSV do Kaggle via API e os armazena no volume do Databricks em `/Volumes/big_data/raw/data/instacart/`. Os arquivos são mantidos em formato original (CSV), sem modificações, como camada de auditoria imutável.

---

### 3. Armazenamento Bruto — BRONZE

**Notebook:** `SRC/ETL/1.BRONZE/update_bronze_tables.ipynb`

**Schema:** `big_data.bronze`

Os CSVs da camada RAW são lidos via Spark e gravados como Delta Tables no schema Bronze. Não são aplicadas transformações de negócio nesta etapa — o objetivo é garantir a persistência dos dados brutos em formato otimizado (Delta), com adição de metadados de controle (`ingestion_timestamp`, `source_file`).

---

### 4. Transformação (Transformation) — SILVER

**Notebook:** `SRC/ETL/2.SILVER/update_silver_tables.ipynb`

**Schema:** `big_data.silver`

Etapa central de qualidade e enriquecimento dos dados. As transformações incluem:

- **Type casting:** Conversão de tipos para os adequados (ex: IDs como inteiros, timestamps corretos).
- **Filtragem de nulos:** Remoção de registros inválidos.
- **Feature engineering:**
  - `is_first_order` — identifica o primeiro pedido de cada cliente.
  - `period_of_day` — classifica o horário do pedido em `morning`, `afternoon`, `evening` ou `night`.
  - `price_band` — categoriza produtos por faixa de preço em `Very Low`, `Low`, `Medium`, `High`, `Premium` ou `Luxury`.
- **Enriquecimento:** Join de `products` com `aisles`, `prices`, `departments` para gerar `products_enriched`.
- **Unificação:** Union de `order_products__prior` e `order_products__train` em uma única tabela `order_products`.

Metadado adicionado: `_silver_timestamp`.

---

### 5. Carregamento e Destino — GOLD

**Notebook:** `SRC/ETL/3.GOLD/analysis_gold_tables.ipynb`

**Schema:** `big_data.gold`

Camada analítica. Os dados da Silver são agregados e transformados em tabelas de negócio prontas para consumo por analistas e ferramentas de visualização.

Grupos de tabelas geradas:

**Sales Analytics**
- Distribuição de vendas por período do dia e hora.
- Taxa de recompra por departamento.

**Customer Analytics**
- Segmentação de clientes por frequência de compra.
- Distribuição de intervalo entre pedidos.

**Product Analytics**
- Performance individual de produtos (volume, recompra).
- Performance por departamento e corredor.

Os resultados ficam disponíveis como tabelas Delta no schema `big_data.gold`, consumíveis diretamente via SQL no Databricks ou por notebooks de visualização.

---

## Checklist AV1

| Etapa | Status |
|---|---|
| Ingestão (RAW) | Finalizado |
| Armazenamento Bronze | Finalizado |
| Transformação Silver | Finalizado |
| Camada Gold (Analítica) | Em progresso |

---

## Divisão de Tarefas

| Membro | Responsabilidade |
|---|---|
| Caio Hirata | Arquitetura Medallion, notebook Bronze |
| Camila Cirne | Transformações Silver, qualidade de dados |
| Diogo Correia | Ingestão RAW, configuração Databricks |
| Flavio Muniz | Camada Gold, métricas de negócio |
| Pedro Coelho | Documentação, diagrama de arquitetura |
| Virna Amaral | Análise exploratória, visualizações |

