# 🎓 E-commerce Analysis

> Projeto de análise de dados de e-commerce utilizando **Databricks + Apache Spark (PySpark)** para processamento distribuído, além de técnicas de exploração, visualização e extração de insights.

![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow?style=for-the-badge)
![Python](https://img.shields.io/badge/Python-3.x-blue?style=for-the-badge&logo=python)
![Spark](https://img.shields.io/badge/Apache%20Spark-Processing-E25A1C?style=for-the-badge&logo=apachespark)
![Databricks](https://img.shields.io/badge/Databricks-Platform-FF3621?style=for-the-badge&logo=databricks)
![Data](https://img.shields.io/badge/Data%20Analysis-Pandas-informational?style=for-the-badge)
![Visualization](https://img.shields.io/badge/Visualization-Matplotlib%20%7C%20Seaborn-orange?style=for-the-badge)


## 📋 Visão Geral do Projeto

O **E-commerce Analysis** é um projeto focado na análise de dados de um ambiente de comércio eletrônico, com o objetivo de extrair insights relevantes sobre vendas, clientes, produtos e logística.

Este projeto pode envolver técnicas de:
- Análise exploratória de dados (EDA)
- Limpeza e tratamento de dados
- Visualização de dados
- Modelagem analítica (opcional)

🔗 **Link do Repositório:** https://github.com/Kal-0/e-commerce_analysis

---

## 🏗️ Arquitetura e Organização do Projeto

### Estrutura de Diretórios
```bash
.
├── src/
│   └── ETL/
│       ├── 0. RAW/
│       │   └── ingest_Instacart_kaggle.ipynb    # Ingestão dos dados brutos do Kaggle
│       ├── 1.BRONZE/
│       │   └── update_bronze_tables.ipynb       # Carga dos dados brutos para o formato Delta (sem transformações)
│       ├── 2.SILVER/
│       │   └── update_silver_tables.ipynb       # Limpeza, tipagem, enriquecimento e qualidade de dados
│       └── 3.GOLD/
│           └── analysis_gold_tables.ipynb       # Agregações, métricas de negócio e tabelas para análise
├── .gitignore                                   # Arquivos e pastas ignorados pelo Git (ex: dados locais)
├── diagram.md                                   # Diagrama Mermaid da Arquitetura Medallion
└── README.md                                    # Documentação principal do projeto
```
### Fluxo de análise
```mermaid
graph LR
    Data[Dataset 📁] --> Cleaning[Limpeza de Dados 🧹]
    Cleaning --> EDA[Análise Exploratória 📊]
    EDA --> Insights[Insights 📈]
    Insights --> Visualization[Visualizações 📉]
```

### Diagrama de Arquitetura
```mermaid
flowchart TD

%% =========================
%% STYLES
%% =========================
classDef raw fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1px,text-align:left;
classDef bronze fill:#efebe9,stroke:#8d6e63,stroke-width:1px,text-align:left;
classDef silver fill:#eceff1,stroke:#78909c,stroke-width:1px,text-align:left;
classDef gold fill:#fff8e1,stroke:#ffca28,stroke-width:1px,text-align:left;

%% =========================
%% RAW LAYER
%% =========================
subgraph RAW["RAW LAYER"]
    direction LR
    
    RAW_INFO["<b>Location:</b> /Volumes/big_data/raw/data/instacart/<br><b>Format:</b> CSV Files (Source Data)"]:::raw
    
    R1["departments.csv"]:::raw
    R2["aisles.csv"]:::raw
    R3["products.csv"]:::raw
    R4["orders.csv"]:::raw
    
    subgraph OP_BLOCK[" "]
        R5["order_products__prior.csv"]:::raw
        R6["order_products__train.csv"]:::raw
    end
    
    %% Conexões internas da RAW
    RAW_INFO --> R1
    RAW_INFO --> R2
    RAW_INFO --> R3
    
    R1 --> R4
    R2 --> OP_BLOCK
    R3 --> OP_BLOCK
end

%% =========================
%% BRONZE LAYER
%% =========================
subgraph BRONZE["BRONZE LAYER"]
    direction LR
    
    BRONZE_INFO["<b>Schema:</b> big_data.bronze<br><b>Notebook:</b> update_bronze_tables<br><b>Process:</b> Raw CSV → Delta Tables<br>(No Transformations)"]:::bronze
    
    B1["departments"]:::bronze
    B2["aisles"]:::bronze
    B3["products"]:::bronze
    B4["orders"]:::bronze
    B5["order_products_prior"]:::bronze
    B6["order_products_train"]:::bronze
    
    %% Conexões internas da BRONZE
    BRONZE_INFO --> B1
    BRONZE_INFO --> B2
    BRONZE_INFO --> B3
    
    B1 --> B4
    B2 --> B5
    B3 --> B6
end

%% =========================
%% SILVER LAYER
%% =========================
subgraph SILVER["SILVER LAYER"]
    direction LR
    
    SILVER_INFO["<b>Schema:</b> big_data.silver<br><b>Notebook:</b> update_silver_tables<br><b>Process:</b> Data Quality + Type Casting<br>+ Feature Engineering + Enrichment"]:::silver
    
    S1["<b>orders</b><br><b>Features Added:</b><br>• is_first_order<br>• period_of_day<br>(morning/afternoon/evening/night)<br><b>Quality:</b><br>• Type casting<br>• Null filtering"]:::silver
    S2["<b>products_enriched</b><br><b>Enrichment:</b><br>• Join with aisles<br>• Join with departments<br>• Trim product names<br><b>Quality:</b><br>• Type casting<br>• Null filtering"]:::silver
    S3["<b>order_products</b><br><b>Combined:</b><br>• prior<br>• train<br><b>Added:</b><br>• dataset flag<br><b>Quality:</b><br>• Type casting<br>• Null filtering"]:::silver
    
    %% Ligações invisíveis para forçar o alinhamento horizontal
    SILVER_INFO ~~~ S1 ~~~ S2 ~~~ S3
end

%% =========================
%% GOLD LAYER
%% =========================
subgraph GOLD["GOLD LAYER"]
    direction LR

    GOLD_INFO["<b>Schema:</b> big_data.gold<br><b>Notebook:</b> analysis_gold_tables<br><b>Process:</b> Business Metrics + Aggregations<br>+ Analysis-Ready Tables"]:::gold

    subgraph SALES["Sales Analytics"]
        direction TB
        G1["<b>sales_by_time_period</b><br>• total_orders<br>• total_items_sold<br>• percentage_orders"]:::gold
        G2["<b>sales_by_hour</b><br>• Hourly patterns<br>• Peak hours"]:::gold
        G3["<b>reorder_analysis_by_department</b><br>• total_items<br>• reordered_items<br>• reorder_rate"]:::gold
        
        %% Força o conteúdo a ficar empilhado verticalmente
        G1 ~~~ G2 ~~~ G3
    end

    subgraph CUSTOMER["Customer Analytics"]
        direction TB
        G4["<b>customer_segmentation</b><br>• New/Occasional/<br>Regular/Frequent/<br>Loyal segments"]:::gold
        G5["<b>customer_purchase_frequency</b><br>• 1-7, 8-14, 15-21,<br>22-30, 30+ days"]:::gold
        
        %% Força o conteúdo a ficar empilhado verticalmente
        G4 ~~~ G5
    end

    subgraph PRODUCT["Product Analytics"]
        direction TB
        G6["<b>product_performance</b><br>• times_ordered<br>• unique_orders<br>• times_reordered<br>• reorder_rate"]:::gold
        G7["<b>department_performance</b><br>• unique_products<br>• total_items_sold<br>• avg_items_per_order"]:::gold
        G8["<b>reorder_analysis_by_aisle</b><br>• total_items<br>• reordered_items<br>• reorder_rate"]:::gold
        
        %% Força o conteúdo a ficar empilhado verticalmente
        G6 ~~~ G7 ~~~ G8
    end

    %% Força os blocos a ficarem lado a lado
    GOLD_INFO ~~~ SALES ~~~ CUSTOMER ~~~ PRODUCT
end

%% =========================
%% CONNECTIONS
%% =========================

%% RAW -> BRONZE

%% BRONZE -> SILVER
BRONZE -->|"Metadata Added:<br>• ingestion_timestamp<br>• source_file<br><br>DATA QUALITY & ENRICHMENT"| SILVER

%% SILVER -> GOLD


%% FLOW LABELS
RAW -->|INGESTION| BRONZE
SILVER -->|"Metadata: _silver_timestamp <br><br>BUSINESS LOGIC & AGGREGATIONS"| GOLD
```

## 🛠️ Stack Tecnológica

* **Linguagem:** Python 3.x
* **Análise de Dados:** Pandas, NumPy
* **Visualização:** Matplotlib, Seaborn
* **Ambiente:** Jupyter Notebook / VS Code
* **(Opcional):** SQL / Power BI / Tableau

## 🚀 Como Executar Localmente

### Pré-requisitos
* Python 3 instalado
* Pip ou Conda

### Passo a Passo (Databricks)

1. **Acesse o Databricks:**
   - Entre na sua workspace do Databricks.

2. **Importe o repositório:**
   - Vá em **Workspace** → **Create** → **Git Folder**
   - Cole o link do repositório:
     ```
     https://github.com/Kal-0/e-commerce_analysis
     ```

3. **Configure o Cluster:**
   - Crie ou selecione um cluster (Runtime padrão já é suficiente para Python e Spark).

4. **Execute os notebooks:**
   - Abra os notebooks dentro do projeto
   - Clique em **Run All** para executar todas as células

5. **Visualize os resultados:**
   - Os outputs (gráficos, tabelas e insights) serão exibidos diretamente no notebook


## 🔄 Pipeline de Análise
O fluxo típico do projeto segue as etapas abaixo:

1. Coleta ou carregamento dos dados
2. Limpeza e pré-processamento
3. Análise exploratória
4. Geração de visualizações
5. Extração de insights

## 🧪 Estratégia de Análise
* Identificação de padrões de compra
* Análise de comportamento de clientes
* Avaliação de performance de vendas
* Análise de logística e entregas

## 📊 Resultados e Insights

## 📊 Evidências

## 📜 Licença
MIT — Livre para uso acadêmico.

## 👥 Colaboradores

| [<img src="https://avatars.githubusercontent.com/u/106926790?v=4" width=115><br><sub>Caio Hirata</sub>](https://github.com/Kal-0) | [<img src="https://avatars.githubusercontent.com/u/28824856?v=4" width=115><br><sub>Camila Cirne</sub>](https://github.com/camilacirne) | [<img src="https://avatars.githubusercontent.com/u/116087739?v=4" width=115><br><sub>Diogo Correia</sub>](https://github.com/DiogoHMC) | [<img src="https://avatars.githubusercontent.com/u/116359369?v=4" width=115><br><sub>Flávio Muniz</sub>](https://github.com/flavio-muniz) | [<img src="https://avatars.githubusercontent.com/u/111138996?v=4" width=115><br><sub>Pedro Coelho</sub>](https://github.com/pedro-coelho-dr) | 
| :---: | :---: | :---: | :---: | :---: |










