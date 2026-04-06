# Medallion Architecture - Instacart ETL Pipeline

## Dataset


## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              RAW LAYER                                       │
│  Location: /Volumes/big_data/raw/data/instacart/                           │
│  Format: CSV Files (Source Data)                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
          ┌─────────▼─────┐  ┌──────▼──────┐  ┌─────▼──────────┐
          │ departments.csv │  │  aisles.csv  │  │  products.csv  │
          └─────────────────┘  └──────────────┘  └────────────────┘
                    │                │                │
          ┌─────────▼─────────┐  ┌─▼────────────────▼──────────────┐
          │    orders.csv      │  │ order_products__prior.csv       │
          └────────────────────┘  │ order_products__train.csv       │
                                  └─────────────────────────────────┘
                                     │
                                     │ INGESTION
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            BRONZE LAYER                                      │
│  Schema: big_data.bronze                                                    │
│  Notebook: update_bronze_tables                                             │
│  Process: Raw CSV → Delta Tables (No Transformations)                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
          ┌──────────────────────────┼──────────────────────────┐
          │                          │                          │
    ┌─────▼──────┐          ┌────────▼────────┐      ┌─────────▼──────────┐
    │ departments │          │     aisles      │      │     products       │
    │             │          │                 │      │                    │
    └─────────────┘          └─────────────────┘      └────────────────────┘
          │                          │                          │
    ┌─────▼──────┐          ┌────────▼─────────────┐  ┌────────▼────────────┐
    │   orders   │          │ order_products_prior │  │ order_products_train│
    └────────────┘          └──────────────────────┘  └─────────────────────┘
          │
          │ Metadata Added:
          │ • ingestion_timestamp
          │ • source_file
          │
          │ DATA QUALITY & ENRICHMENT
          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            SILVER LAYER                                      │
│  Schema: big_data.silver                                                    │
│  Notebook: update_silver_tables                                             │
│  Process: Data Quality + Type Casting + Feature Engineering + Enrichment   │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
          ┌──────────────────────────┼──────────────────────────┐
          │                          │                          │
    ┌─────▼──────────────┐  ┌────────▼────────────────┐  ┌────▼───────────┐
    │      orders        │  │  products_enriched      │  │ order_products │
    │                    │  │                         │  │                │
    │ Features Added:    │  │ Enrichment:             │  │ Combined:      │
    │ • is_first_order   │  │ • Join with aisles      │  │ • prior        │
    │ • period_of_day    │  │ • Join with departments │  │ • train        │
    │   (morning/        │  │ • Trim product names    │  │                │
    │    afternoon/      │  │                         │  │ Added:         │
    │    evening/night)  │  │                         │  │ • dataset flag │
    │                    │  │                         │  │                │
    │ Quality:           │  │ Quality:                │  │ Quality:       │
    │ • Type casting     │  │ • Type casting          │  │ • Type casting │
    │ • Null filtering   │  │ • Null filtering        │  │ • Null filter  │
    └────────────────────┘  └─────────────────────────┘  └────────────────┘
          │
          │ Metadata: _silver_timestamp
          │
          │ BUSINESS LOGIC & AGGREGATIONS
          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                             GOLD LAYER                                       │
│  Schema: big_data.gold                                                      │
│  Notebook: analysis_gold_tables                                             │
│  Process: Business Metrics + Aggregations + Analysis-Ready Tables          │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
          ┌──────────────────────────┼──────────────────────────┐
          │                          │                          │
    ┌─────▼────────────────┐  ┌──────▼─────────────────┐  ┌───▼───────────────┐
    │ Sales Analytics      │  │ Customer Analytics     │  │ Product Analytics │
    └──────────────────────┘  └────────────────────────┘  └───────────────────┘
          │                          │                          │
          ├─► sales_by_time_period   ├─► customer_segmentation ├─► product_performance
          │   • total_orders          │   • New/Occasional/     │   • times_ordered
          │   • total_items_sold      │     Regular/Frequent/   │   • unique_orders
          │   • percentage_orders     │     Loyal segments      │   • times_reordered
          │                           │                         │   • reorder_rate
          ├─► sales_by_hour          ├─► customer_purchase_    │
          │   • Hourly patterns       │    frequency            ├─► department_performance
          │   • Peak hours            │   • 1-7, 8-14, 15-21,   │   • unique_products
          │                           │     22-30, 30+ days     │   • total_items_sold
          │                           │                         │   • avg_items_per_order
          │                           │                         │
          ├─► reorder_analysis_      │                         ├─► reorder_analysis_by_aisle
              by_department           │                             • total_items
              • total_items           │                             • reordered_items
              • reordered_items       │                             • reorder_rate
              • reorder_rate          │
                                      │
```

## Layer Details

### RAW Layer
- Location: /Volumes/big_data/raw/data/instacart/
- Format: CSV files
- Files: 
  - departments.csv
  - aisles.csv
  - products.csv
  - orders.csv
  - order_products__prior.csv
  - order_products__train.csv

### BRONZE Layer
- Schema: big_data.bronze
- Notebook: update_bronze_tables
- Format: Delta Tables
- Transformations:
  - CSV → Delta format conversion
  - Add ingestion_timestamp
  - Add source_file reference
- Tables: departments, aisles, products, orders, order_products_prior, order_products_train

### SILVER Layer
- Schema: big_data.silver
- Notebook: update_silver_tables
- Format: Delta Tables (Cleaned & Enriched)
- Transformations:
  - Data Quality: Type casting, null filtering, data validation
  - Feature Engineering: 
    - is_first_order (boolean flag)
    - period_of_day (morning/afternoon/evening/night)
  - Enrichment: Join products with aisles and departments
  - Consolidation: Merge prior + train datasets
- Tables: orders, products_enriched, order_products

### GOLD Layer
- Schema: big_data.gold
- Notebook: analysis_gold_tables
- Format: Delta Tables (Aggregated Metrics)
- Categories:
  Sales Analytics:
  - sales_by_time_period - Orders by period of day
  - sales_by_hour - Hourly sales patterns
  - reorder_analysis_by_department - Department-level reorder rates
  - reorder_analysis_by_aisle - Aisle-level reorder rates
  
  Customer Analytics:
  - customer_segmentation - Customer loyalty segments
  - customer_purchase_frequency - Time between orders distribution
  
  Product Analytics:
  - product_performance - Top products by orders and reorders
  - department_performance - Department sales metrics


## Data Flow Summary

1. Ingestion (RAW → BRONZE)
   - Read CSV files from volume storage
   - Convert to Delta format with metadata
   - No business logic applied

2. Transformation (BRONZE → SILVER)
   - Apply data quality rules
   - Type casting and validation
   - Feature engineering (time periods, first order flags)
   - Enrich products with category information
   - Combine prior and train datasets

3. Aggregation (SILVER → GOLD)
   - Calculate business KPIs
   - Create customer segmentation
   - Analyze sales patterns by time
   - Measure product and category performance
   - Generate reorder analytics


## Technologies Used

- Storage: Databricks Volumes (Unity Catalog)
- Format: Delta Lake
- Processing: PySpark
- Orchestration: Databricks Notebooks
