## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              RAW LAYER                                      │
│  Location: /Volumes/big_data/raw/data/instacart/                            │
│  Format: CSV Files (Source Data)                                            │
│  Notebook: ingest_Instacart_kaggle                                          │
│  Process: Kaggle API → Download & Unzip → Copy to Unity Catalog Volume      │
└─────────────────────────────────────────────────────────────────────────────┘
                                  │
                ┌─────────────────┼─────────────────┐
                │                 │                 │
      ┌─────────▼───────┐  ┌──────▼───────┐  ┌─────▼──────────┐
      │ departments.csv │  │  aisles.csv  │  │  products.csv  │
      └─────────────────┘  └──────────────┘  └────────────────┘
                │               │                │
      ┌─────────▼──────────┐  ┌─▼────────────────▼──────────────┐
      │    orders.csv      │  │ order_products__prior.csv       │
      └────────────────────┘  │ order_products__train.csv       │
                              └─────────────────────────────────┘
                                                 │
                                ┌────────────────┴─────────────────┐
                                │                                  │
                                │                    ┌─────────────▼──────────┐
                                │                    │    prices.csv          │
                                │                    │ (Product Pricing Data) │
                                │                    └────────────────────────┘
                                │
                                │ INGESTION
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            BRONZE LAYER                                     │
│  Schema: big_data.bronze                                                    │
│  Notebook: update_bronze_tables                                             │
│  Process: Raw CSV → Delta Tables (No Transformations)                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
        ┌───────────────────────────┼─────────────────────────┐
        │                           │                         │
  ┌─────▼───────┐          ┌────────▼────────┐      ┌─────────▼──────────┐
  │ departments │          │     aisles      │      │     products       │
  │             │          │                 │      │                    │
  └─────────────┘          └─────────────────┘      └────────────────────┘
        │                          │                         │
  ┌─────▼──────┐          ┌────────▼─────────────┐  ┌────────▼─────────────┐
  │   orders   │          │ order_products_prior │  │ order_products_train │
  └────────────┘          └──────────────────────┘  └──────────────────────┘
        │
  ┌─────▼──────┐
  │   prices   │
  └────────────┘
      │
      │ Metadata Added:
      │ • ingestion_timestamp
      │ • source_file
      │
      │ DATA QUALITY & ENRICHMENT
      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            SILVER LAYER                                     │
│  Schema: big_data.silver                                                    │
│  Notebook: update_silver_tables                                             │
│  Process: Data Quality + Type Casting + Feature Engineering + Enrichment    │
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
  │   (morning/        │  │ • Join with prices      │  │                │
  │    afternoon/      │  │ • price_band            │  │ Added:         │
  │    evening/night)  │  │   (Very Low/Low/        │  │ • dataset flag │
  │                    │  │    Medium/High/         │  │                │
  │ Quality:           │  │    Premium/Luxury)      │  │ Quality:       │
  │ • Type casting     │  │ • Trim product names    │  │ • Type casting │
  │ • Null filtering   │  │                         │  │ • Null filter  │
  │                    │  │ Quality:                │  │                │
  │                    │  │ • Type casting          │  │                │
  │                    │  │ • Null filtering        │  │                │
  └────────────────────┘  └─────────────────────────┘  └────────────────┘
      │
      │ Metadata: _silver_timestamp
      │
      │ BUSINESS LOGIC & AGGREGATIONS
      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                             GOLD LAYER                                      │
│  Schema: big_data.gold                                                      │
│  Notebook: analysis_gold_tables                                             │
│  Process: Business Metrics + Aggregations + Analysis-Ready Tables           │
└─────────────────────────────────────────────────────────────────────────────┘
                   │
      ┌──────────────────────────┼──────────────────────────┐
      │                          │                          │
  ┌─────▼────────────────┐  ┌──────▼─────────────────┐  ┌───▼───────────────┐
  │ Sales Analytics      │  │ Customer Analytics     │  │ Product Analytics │
  └──────────────────────┘  └────────────────────────┘  └───────────────────┘
      │                           │                          │
      ├─► sales_by_time_period    ├─► customer_segmentation ├─► product_performance
      │   • total_orders          │   • New/Occasional/     │   • times_ordered
      │   • total_items_sold      │     Regular/Frequent/   │   • unique_orders
      │   • percentage_orders     │     Loyal segments      │   • times_reordered
      │                           │                         │   • reorder_rate
      ├─► sales_by_hour           ├─► customer_purchase_    │
      │   • Hourly patterns       │    frequency            ├─► department_performance
      │   • Peak hours            │   • 1-7, 8-14, 15-21,  │   • unique_products
      │                           │     22-30, 30+ days     │   • total_items_sold
      │                           │                         │   • unique_orders
      ├─► reorder_analysis_       │                         │   • reordered_items
      │   by_department           │                         │   • reorder_rate
      │   • total_items           │                         │   • avg_items_per_order
      │   • reordered_items       │                         │
      │   • reorder_rate          │                         ├─► reorder_analysis_by_aisle
      │                           │                         │   • total_items
      │                           │                         │   • reordered_items
      │                           │                         │   • reorder_rate
```
