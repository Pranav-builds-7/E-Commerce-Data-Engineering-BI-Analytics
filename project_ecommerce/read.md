# Ecommerce Data Pipeline

A Databricks and PySpark project that processes ecommerce data through a medallion architecture:

- **Bronze** stores source data with ingestion metadata.
- **Silver** cleans and standardizes the data.
- **Gold** creates dimension and fact tables for reporting and analytics.

## Project structure

```text
project_ecommerce/
├── setup/
│   └── setup.py
├── medallion_processing_dim/
│   ├── 1_dim_bronze.py
│   ├── 1_dim_silver.py
│   └── 1_dim_golden.py
└── medallion_processing_fact/
    ├── 1_fact_bronze.py
    ├── 2_fact_silver.py
    └── 3_fact_gold.py
```

## Requirements

Run this project in a Databricks workspace with:

- PySpark
- Unity Catalog enabled
- Access to the `ecommerce` catalog
- Source CSV files available in the Databricks volume paths used by the notebooks

The project uses Databricks notebook features such as `%sql`, `display()`, and `spark`.

## Source data

The notebooks expect CSV files under:

```text
/Volumes/ecommerce/source_data/raw/
```

Expected folders:

```text
brands/
category/
products/
customers/
date/
order_items/landing/
```

The order items source is expected to include fields for order date and timestamp, customer and product IDs, quantity, currency, unit price, discount, tax, sales channel, and coupon code.

## Data layers and output tables

### Bronze

Raw CSV data is loaded into Delta tables:

```text
ecommerce.bronze.brz_brands
ecommerce.bronze.brz_category
ecommerce.bronze.brz_products
ecommerce.bronze.brz_customers
ecommerce.bronze.brz_calendar
ecommerce.bronze.brz_order_items
```

### Silver

The silver notebooks clean and standardize source data, including product attributes, customer records, calendar fields, and order item values.

```text
ecommerce.silver.slv_brands
ecommerce.silver.slv_category
ecommerce.silver.slv_products
ecommerce.silver.slv_customers
ecommerce.silver.slv_calendar
ecommerce.silver.slv_order_items
```

### Gold

The gold notebooks create reporting tables:

```text
ecommerce.gold.gld_dim_products
ecommerce.gold.gld_dim_customers
ecommerce.gold.gld_dim_date
ecommerce.gold.gld_fact_order_items
```

The fact table includes calculated gross, discount, and net amounts, plus an INR-converted amount based on the exchange rates defined in the gold fact notebook.

## Setup and run order

1. Import the project notebooks into Databricks.
2. Run `setup/setup.py` to create the `ecommerce` catalog and its `bronze`, `silver`, and `gold` schemas.
3. Place the source CSV files in the expected volume folders.
4. Run the dimension bronze notebook, then the dimension silver notebook, then the dimension gold notebook.
5. Run the fact bronze notebook, then the fact silver notebook, then the fact gold notebook.

The gold fact table uses cleaned silver order data. The gold dimension tables are built from the silver dimension tables.

## Important notes

- The notebooks currently write tables in **overwrite** mode. Rerunning a notebook replaces the corresponding table contents.
- Exchange rates are hardcoded in the gold fact notebook and should be reviewed before using INR amounts for financial reporting.
- Check source paths, CSV column names, date formats, currency values, and business rules before running this pipeline with new data.
- Validate table counts and data quality after each layer completes.
