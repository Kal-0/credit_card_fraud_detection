# E-commerce Analysis — Instacart Market Basket

Projeto de Big Data para a disciplina de Fundamentos de Big Data.
Pipeline completo de ingestão, transformação e análise de dados de e-commerce, executado integralmente no **Databricks** com arquitetura **Medallion** (Raw → Bronze → Silver → Gold).

**Repositório:** https://github.com/Kal-0/e-commerce_analysis

---

## Descrição

O projeto analisa o dataset público [Instacart Market Basket Analysis](https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis) para extrair insights sobre comportamento de compra, performance de produtos e segmentação de clientes.

O pipeline é construído sobre **Apache Spark (PySpark)** no Databricks, organizando os dados em camadas Medallion com tabelas Delta Lake.

---

## Fonte de Dados

**Dataset:** Instacart Market Basket Analysis (Kaggle)

Arquivos CSV ingeridos na camada RAW:

| Arquivo | Descrição |
|---|---|
| `orders.csv` | Pedidos realizados pelos clientes |
| `order_products__prior.csv` | Produtos de pedidos anteriores |
| `order_products__train.csv` | Produtos de pedidos de treino |
| `products.csv` | Catálogo de produtos |
| `prices.csv` | Preços sintéticos dos produtos |
| `aisles.csv` | Corredores do supermercado |
| `departments.csv` | Departamentos do supermercado |

---

## Stack Tecnológica

| Camada | Tecnologia |
|---|---|
| Plataforma | Databricks |
| Processamento | Apache Spark / PySpark |
| Armazenamento | Delta Lake |
| Linguagem | Python 3.x |
| Visualização | Databricks `display()` |
| Formato de dados | CSV (Raw), Delta Tables (Bronze/Silver/Gold) |

---

## Estrutura do Repositório

```
.
├── SRC/
│   └── ETL/
│       ├── 0.RAW/
│       │   └── ingest_Instacart_kaggle.ipynb       # Ingestão dos dados brutos do Kaggle
│       ├── 1.BRONZE/
│       │   └── update_bronze_tables.ipynb          # CSV bruto -> Delta Tables (sem transformações)
│       ├── 2.SILVER/
│       │   └── update_silver_tables.ipynb          # Limpeza, tipagem, enriquecimento
│       └── 3.GOLD/
│           └── analysis_gold_tables.ipynb          # Agregações e métricas de negócio
├── docs/
│   ├── arquitetura.md                              # Diagrama Medallion e detalhes da arquitetura
│   ├── diagram.md                                  # Diagrama ASCII da arquitetura e fluxo de dados
│   └── pipeline.md                                 # Descrição detalhada de cada etapa do pipeline
├── .gitignore
└── README.md
```

---

## Como Executar no Databricks

1. Acesse sua workspace do Databricks.
2. Vá em **Workspace** → **Create** → **Git Folder** e cole o link do repositório.
3. Crie ou selecione um cluster (Runtime com Spark).
4. Execute os notebooks na ordem das camadas: RAW → BRONZE → SILVER → GOLD.
5. Os resultados ficam disponíveis como tabelas Delta no schema `big_data.*`.

Mais detalhes em [docs/arquitetura.md](docs/arquitetura.md).

---

## Documentação

- [Arquitetura Medallion e Diagrama](docs/arquitetura.md)
- [Diagrama ASCII do Fluxo de Dados](docs/diagram.md)
- [Descrição do Pipeline](docs/pipeline.md)

---

## Colaboradores

| [<img src="https://avatars.githubusercontent.com/u/106926790?v=4" width=115><br><sub>Caio Hirata</sub>](https://github.com/Kal-0) | [<img src="https://avatars.githubusercontent.com/u/28824856?v=4" width=115><br><sub>Camila Cirne</sub>](https://github.com/camilacirne) | [<img src="https://avatars.githubusercontent.com/u/116087739?v=4" width=115><br><sub>Diogo Correia</sub>](https://github.com/DiogoHMC) | [<img src="https://avatars.githubusercontent.com/u/116359369?v=4" width=115><br><sub>Flavio Muniz</sub>](https://github.com/flavio-muniz) | [<img src="https://avatars.githubusercontent.com/u/111138996?v=4" width=115><br><sub>Pedro Coelho</sub>](https://github.com/pedro-coelho-dr) | [<img src="https://avatars.githubusercontent.com/virnaamaral?v=4" width=115><br><sub>Virna Amaral</sub>](https://github.com/virnaamaral) |
| :---: | :---: | :---: | :---: | :---: | :---: |

---

## Licença

MIT — Livre para uso acadêmico.
