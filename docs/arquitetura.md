# Arquitetura do Pipeline — Medallion Architecture

## Visao Geral

O pipeline segue a arquitetura Medallion, organizada em quatro camadas progressivas de refinamento dos dados. Todo o processamento ocorre no **Databricks** com **Apache Spark (PySpark)** e armazenamento em **Delta Lake**.

```
Kaggle (CSV) → RAW → BRONZE → SILVER → GOLD
```

---

## Camadas

### RAW
- **Localizacao:** `/Volumes/big_data/raw/data/instacart/`
- **Formato:** CSV original do Kaggle
- **Notebook:** `SRC/ETL/0.RAW/ingest_Instacart_kaggle.ipynb`
- **Descricao:** Ingestao dos arquivos brutos do Kaggle para o volume do Databricks, sem qualquer transformacao. Ponto de origem imutavel dos dados.

### BRONZE
- **Schema:** `big_data.bronze`
- **Formato:** Delta Tables
- **Notebook:** `SRC/ETL/1.BRONZE/update_bronze_tables.ipynb`
- **Descricao:** Carga direta dos CSVs da camada RAW para tabelas Delta, sem transformacoes de negocio. Adiciona metadados de controle:
  - `ingestion_timestamp`
  - `source_file`

Tabelas geradas: `departments`, `aisles`, `products`, `orders`, `order_products_prior`, `order_products_train`

### SILVER
- **Schema:** `big_data.silver`
- **Formato:** Delta Tables
- **Notebook:** `SRC/ETL/2.SILVER/update_silver_tables.ipynb`
- **Descricao:** Camada de qualidade e enriquecimento. Aplica limpeza, tipagem correta, joins e feature engineering.

Tabelas geradas e transformacoes aplicadas:

| Tabela | Transformacoes |
|---|---|
| `orders` | Type casting, filtragem de nulos, criacao de `is_first_order` e `period_of_day` (morning/afternoon/evening/night) |
| `products_enriched` | Join com `aisles` e `departments`, trim de nomes, type casting, filtragem de nulos |
| `order_products` | Union de `prior` + `train`, adicao de flag `dataset`, type casting, filtragem de nulos |

Metadado adicionado: `_silver_timestamp`

### GOLD
- **Schema:** `big_data.gold`
- **Formato:** Delta Tables
- **Notebook:** `SRC/ETL/3.GOLD/analysis_gold_tables.ipynb`
- **Descricao:** Camada analitica com agregacoes e metricas de negocio prontas para consumo.

Tabelas geradas:

**Sales Analytics**
- `sales_by_time_period` — total de pedidos, itens vendidos e percentual por periodo do dia
- `sales_by_hour` — padroes de compra por hora, identificacao de horarios de pico
- `reorder_analysis_by_department` — taxa de recompra por departamento

**Customer Analytics**
- `customer_segmentation` — segmentos: New / Occasional / Regular / Frequent / Loyal
- `customer_purchase_frequency` — distribuicao por intervalo de dias entre compras

**Product Analytics**
- `product_performance` — vezes comprado, pedidos unicos, taxa de recompra
- `department_performance` — produtos unicos, total vendido, media de itens por pedido
- `reorder_analysis_by_aisle` — taxa de recompra por corredor

---

## Diagrama

```mermaid
flowchart TD

classDef raw fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1px;
classDef bronze fill:#efebe9,stroke:#8d6e63,stroke-width:1px;
classDef silver fill:#eceff1,stroke:#78909c,stroke-width:1px;
classDef gold fill:#fff8e1,stroke:#ffca28,stroke-width:1px;

subgraph RAW["RAW LAYER — /Volumes/big_data/raw/data/instacart/"]
    direction LR
    R1["departments.csv"]:::raw
    R2["aisles.csv"]:::raw
    R3["products.csv"]:::raw
    R4["orders.csv"]:::raw
    R5["order_products__prior.csv"]:::raw
    R6["order_products__train.csv"]:::raw
end

subgraph BRONZE["BRONZE LAYER — big_data.bronze"]
    direction LR
    B1["departments"]:::bronze
    B2["aisles"]:::bronze
    B3["products"]:::bronze
    B4["orders"]:::bronze
    B5["order_products_prior"]:::bronze
    B6["order_products_train"]:::bronze
end

subgraph SILVER["SILVER LAYER — big_data.silver"]
    direction LR
    S1["orders\n+ is_first_order\n+ period_of_day"]:::silver
    S2["products_enriched\n+ aisle + department"]:::silver
    S3["order_products\n(prior + train unificados)"]:::silver
end

subgraph GOLD["GOLD LAYER — big_data.gold"]
    direction LR
    subgraph SALES["Sales Analytics"]
        G1["sales_by_time_period"]:::gold
        G2["sales_by_hour"]:::gold
        G3["reorder_analysis_by_department"]:::gold
    end
    subgraph CUSTOMER["Customer Analytics"]
        G4["customer_segmentation"]:::gold
        G5["customer_purchase_frequency"]:::gold
    end
    subgraph PRODUCT["Product Analytics"]
        G6["product_performance"]:::gold
        G7["department_performance"]:::gold
        G8["reorder_analysis_by_aisle"]:::gold
    end
end

RAW -->|"Ingestao — CSV to Delta"| BRONZE
BRONZE -->|"Data Quality + Enrichment"| SILVER
SILVER -->|"Business Metrics + Aggregations"| GOLD
```

---

## Estrutura de Diretorios

```
SRC/
└── ETL/
    ├── 0.RAW/
    │   └── ingest_Instacart_kaggle.ipynb
    ├── 1.BRONZE/
    │   └── update_bronze_tables.ipynb
    ├── 2.SILVER/
    │   └── update_silver_tables.ipynb
    └── 3.GOLD/
        └── analysis_gold_tables.ipynb
```

---

## Stack Tecnologica

| Componente | Tecnologia |
|---|---|
| Plataforma | Databricks |
| Processamento distribuido | Apache Spark / PySpark |
| Armazenamento / ACID | Delta Lake |
| Linguagem | Python 3.x |
| Formato de origem | CSV |
| Formato de destino | Delta Tables |
| Visualizacao | Matplotlib, Seaborn |
| Controle de versao | Git / GitHub |
