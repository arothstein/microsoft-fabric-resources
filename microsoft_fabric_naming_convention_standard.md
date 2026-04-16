# Microsoft Fabric Enterprise Naming Convention Standard

**Version:** 1.0  
**Effective Date:** April 2026  
**Classification:** Internal — Governance Standard

---

## 1. Purpose and Scope

This standard defines the enterprise naming convention for all objects within the organization's Microsoft Fabric environment. It applies to every artifact created across all Fabric workloads—Data Factory, Data Engineering, Data Science, Data Warehouse, Real-Time Intelligence, Power BI, and Data Activator.

The goals of this standard are to:

- Guarantee **uniqueness** of every object name within its scope
- Provide **self-documenting** names that communicate what an object is, what it belongs to, and where it sits in the data lifecycle
- Enable any team member—including new hires—to quickly orient themselves in the environment
- Support **automation**, CI/CD (Deployment Pipelines), lineage tracking, and governance at scale

> **Note on Official Guidance:** As of this writing, Microsoft has not published a dedicated Fabric naming convention. However, Microsoft's [Cloud Adoption Framework — Define Your Naming Convention](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming) establishes the foundational pattern for Azure resources: `{resource-type}-{workload/purpose}-{environment}-{region}-{instance}`. Microsoft's [Workspace Planning guidance](https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning) further recommends short, descriptive workspace names with standard prefixes and suffixes. This standard adapts those principles to the specific needs of Fabric artifacts, drawing also on widely-referenced frameworks from Advancing Analytics, XTIVIA, and other recognized practitioners.

---

## 2. Foundational Rules

These rules apply universally to every named object in the Fabric environment.

### 2.1 Character Set

| Rule | Detail |
|---|---|
| **Case** | All lowercase. This aligns with Microsoft Azure's recommendation and avoids issues with case-sensitive systems (e.g., Spark, Delta Lake, SQL endpoints). |
| **Separator** | Use underscores (`_`) to separate naming segments. Lakehouses—the most restrictive artifact—only allow alphanumeric characters and underscores, and the first character must be a letter. Adopting the most restrictive rule across all artifact types ensures consistency. |
| **Prohibited characters** | No spaces, hyphens, periods, parentheses, or other special characters in artifact names. (Workspaces are an exception—see §3.1.) |
| **Length** | Keep names under 80 characters. Fabric UIs truncate long names, making them hard to scan. |
| **Abbreviations** | Use an approved abbreviation registry (see §5). If you abbreviate a term once, abbreviate it everywhere. Never invent one-off abbreviations. |

### 2.2 Naming Formula

Every artifact name is assembled from a set of **segments** in a fixed order. Not every segment is required for every artifact type, but the order never changes:

```
[project]_[experience]_[type]_[index]_[layer]_[purpose]_[suffix]
```

| # | Segment | Required? | Description |
|---|---|---|---|
| 1 | **Project** | Usually | The project, domain, or business initiative (e.g., `sales`, `hr`, `fin`, `supply`). Omit only when the workspace itself already scopes to a single project. |
| 2 | **Experience** | Optional | The Fabric workload/experience group (e.g., `df`, `de`, `ds`, `dw`, `pbi`). Useful for sorting in mixed workspaces; can be omitted if your workspace strategy already separates by workload. |
| 3 | **Type** | **Mandatory** | The artifact type abbreviation (e.g., `pl`, `nb`, `lh`, `wh`, `rpt`). This is the only truly mandatory segment because it identifies *what* the object is. |
| 4 | **Index** | Optional | A three-digit ordering number (`100`, `200`, `300`). Strongly recommended for pipelines, notebooks, and lakehouses to express sequencing or dependency order. |
| 5 | **Layer** | Recommended | The data processing stage: `bronze`, `silver`, `gold` (or your organization's equivalent such as `raw`, `cleansed`, `curated`). Strongly recommended for lakehouses, warehouses, notebooks, and pipelines. |
| 6 | **Purpose** | Usually | A brief, descriptive label for the object's business function (e.g., `customer_orders`, `daily_sales_refresh`, `churn_prediction`). |
| 7 | **Suffix** | Optional | Additional qualifier when needed, such as an algorithm abbreviation for ML models (`xgb`, `rf`, `lr`) or an instance number (`01`, `02`). |

### 2.3 Why This Order?

The segment order is deliberate. Placing **type** early ensures that artifacts of the same kind sort together in the workspace list. Placing **index** and **layer** before **purpose** creates a natural reading order that mirrors the data flow: *what it is → where in the pipeline → what it does*.

---

## 3. Naming Patterns by Object Category

### 3.1 Workspaces

Workspaces are the top-level organizational container. Because workspace names are visible to all users—including business stakeholders—they follow a slightly different convention.

**Pattern:**  
```
[Domain]-[Project/Workload] [Environment Suffix]
```

**Rules:**

- Use **Title Case with hyphens** for readability (workspaces permit spaces and special characters, but hyphens keep names URL-friendly).
- Put the most important word first (Fabric UIs may truncate).
- Append an environment suffix in brackets **only for non-production**: `[Dev]`, `[Test]`, `[UAT]`. Leave production clean with no suffix—this is the name business users will see.
- Omit the words "Workspace," "Fabric," or "Power BI" as they are redundant.
- Include the organization name only if external users will access the workspace.

**Examples:**

| Workspace Name | Explanation |
|---|---|
| `Finance-Quarterly-Reporting` | Production workspace for the Finance team's quarterly reports |
| `Finance-Quarterly-Reporting [Dev]` | Development counterpart |
| `Supply-Chain-Analytics` | Production workspace for supply chain |
| `HR-People-Analytics [Test]` | Test environment for HR analytics |

> **Alignment with Microsoft guidance:** Microsoft's workspace planning documentation recommends short yet descriptive names with a standard prefix for grouping (e.g., `FIN-`) and a suffix for non-production environments.

### 3.2 Lakehouses

Lakehouses are the most restrictive artifact (alphanumeric + underscores only, must start with a letter). They also benefit the most from the medallion layer indicator.

**Pattern:**  
```
[project]_lh_[index]_[layer]_[purpose]
```

**Examples:**

| Name | Reading |
|---|---|
| `sales_lh_100_bronze_erp_orders` | Sales project, Lakehouse, first stage, bronze layer, ERP order data |
| `sales_lh_200_silver_orders` | Sales project, Lakehouse, second stage, silver layer, cleansed orders |
| `sales_lh_300_gold_sales_mart` | Sales project, Lakehouse, third stage, gold layer, analytics-ready mart |
| `hr_lh_100_bronze_hris_employees` | HR project, Lakehouse, first stage, bronze, HRIS employee data |

> **Why 100/200/300?** Three-digit spacing lets you insert intermediate stages later (e.g., `150` for a validation step) without renumbering everything. This recommendation comes from the Advancing Analytics framework and has been widely adopted.

### 3.3 Data Warehouses

**Pattern:**  
```
[project]_wh_[layer]_[purpose]
```

**Examples:**

| Name | Reading |
|---|---|
| `sales_wh_gold_sales_analytics` | Gold-layer warehouse for sales analytics |
| `fin_wh_gold_financial_reporting` | Financial reporting warehouse |

### 3.4 Pipelines (Data Factory)

**Pattern:**  
```
[project]_pl_[index]_[layer]_[purpose]
```

**Examples:**

| Name | Reading |
|---|---|
| `sales_pl_100_bronze_ingest_erp_orders` | First pipeline—ingests ERP order data into bronze |
| `sales_pl_200_silver_transform_orders` | Second pipeline—transforms orders into silver |
| `sales_pl_300_gold_load_sales_mart` | Third pipeline—loads the gold sales mart |
| `sales_pl_000_orchestrator_daily` | Master orchestrator pipeline (index `000` indicates parent) |

### 3.5 Dataflows Gen2

**Pattern:**  
```
[project]_dfl_[index]_[layer]_[purpose]
```

**Examples:**

| Name | Reading |
|---|---|
| `sales_dfl_100_bronze_ingest_crm_contacts` | Dataflow ingesting CRM contacts into bronze |
| `hr_dfl_200_silver_transform_employee_dim` | Dataflow building the employee dimension in silver |

### 3.6 Notebooks

**Pattern:**  
```
[project]_nb_[index]_[layer]_[purpose]
```

**Examples:**

| Name | Reading |
|---|---|
| `sales_nb_100_bronze_raw_data_validation` | Notebook for validating raw data in bronze |
| `sales_nb_200_silver_transform_orders` | Notebook transforming orders into silver |
| `sales_nb_900_utility_schema_migration` | Utility notebook (index `900` indicates non-pipeline utility) |

### 3.7 Semantic Models (Datasets)

Semantic models sit at the boundary between engineering and business consumption. They should still follow the convention but can include a more business-friendly purpose segment.

**Pattern:**  
```
[project]_sm_[purpose]
```

**Examples:**

| Name | Reading |
|---|---|
| `sales_sm_sales_performance` | Semantic model powering sales performance reports |
| `fin_sm_budget_vs_actuals` | Semantic model for budget analysis |

### 3.8 Reports and Dashboards

Reports and dashboards are the primary business-facing artifacts. Microsoft community experts (and common sense) recommend keeping these **business-friendly** without heavy prefixes, since end users interact with them directly.

**Pattern:**  
```
[project]_rpt_[purpose]
```

Or, for dashboards intended for embedded or app distribution, omit the prefix entirely and use a clear business name.

**Examples:**

| Name | Context |
|---|---|
| `sales_rpt_executive_dashboard` | Internal technical name in the workspace |
| `Executive Sales Dashboard` | Display name in a Power BI App (no prefix needed) |
| `fin_rpt_monthly_close_summary` | Finance monthly close report |

> **Guidance:** The Fabric community correctly notes that reports, apps, and dashboards visible to end users should prioritize clarity over convention enforcement. Apply prefixes in the workspace; let the App name be plain English.

### 3.9 Spark Job Definitions

**Pattern:**  
```
[project]_sj_[index]_[layer]_[purpose]
```

### 3.10 ML Experiments and Models (Data Science)

**Pattern for Experiments:**  
```
[project]_exp_[purpose]
```

**Pattern for Models:**  
```
[project]_mdl_[purpose]_[algorithm_suffix]
```

**Algorithm Suffix Registry:**

| Algorithm | Suffix |
|---|---|
| Decision Tree | `dt` |
| Random Forest | `rf` |
| Logistic Regression | `lor` |
| Linear Regression | `lir` |
| XGBoost | `xgb` |
| LightGBM | `lgbm` |
| Neural Network | `nn` |
| Support Vector Machine | `svm` |

**Examples:**

| Name | Reading |
|---|---|
| `sales_exp_churn_prediction` | Experiment exploring customer churn models |
| `sales_mdl_churn_prediction_xgb` | XGBoost churn prediction model |
| `sales_mdl_churn_prediction_lgbm` | LightGBM variant for comparison |

### 3.11 Eventstreams and KQL Databases (Real-Time Intelligence)

**Pattern:**  
```
[project]_es_[purpose]       (Eventstream)
[project]_kdb_[purpose]      (KQL Database)
[project]_kqs_[purpose]      (KQL Queryset)
```

### 3.12 Data Activator (Reflex)

**Pattern:**  
```
[project]_rfx_[purpose]
```

---

## 4. Lakehouse Tables, Columns, Files, and Folder Structure

The naming standard for artifacts (§3) gets objects organized in the workspace. This section goes one level deeper—into the data objects *inside* each lakehouse. This is where the day-to-day work of data engineers and analysts lives, and where poor naming causes the most pain: ambiguous table names, broken columns, ungovernable file sprawl, and queries that nobody can read six months later.

> **Key Technical Constraint:** Microsoft Fabric Lakehouse uses Delta Lake as its default table format. Delta tables (and their underlying Parquet files) do not support spaces or most special characters in table names or column names. Fabric's Dataflow Gen2 will silently drop columns that contain unsupported characters unless you explicitly fix them during ingestion. Always design names to be compatible with the most restrictive layer of the stack.

---

### 4.1 Lakehouse Physical Structure Overview

Every Fabric Lakehouse exposes two top-level storage areas:

| Area | Purpose | Format Guidance |
|---|---|---|
| `/Tables` | **Managed area.** Delta tables live here. Fabric auto-discovers them and exposes them through the SQL analytics endpoint. | Delta format (default and strongly recommended). Tables written here appear automatically in the Lakehouse explorer. |
| `/Files` | **Unmanaged area.** A landing zone for raw files of any format—CSV, Parquet, JSON, images, Excel, etc. | Any format. Files here are *not* auto-surfaced as tables. Use this for raw ingestion staging, archives, and non-tabular data. |

When using a **schema-enabled lakehouse** (now the default for new lakehouses), you also get named schemas under `/Tables` (e.g., `/Tables/dbo/`, `/Tables/staging/`, `/Tables/curated/`). Schema names follow the same character rules as table names: letters, numbers, and underscores only.

> **Microsoft guidance:** Fabric documentation recommends using four-part naming (`workspace.lakehouse.schema.table`) in Spark SQL even for non-schema lakehouses, so queries are forward-compatible if schemas are enabled later.

---

### 4.2 Table Naming — General Rules

All Delta table names within a lakehouse must follow these rules:

| Rule | Detail |
|---|---|
| **Case** | All lowercase. Delta table names are case-insensitive in Spark by default, but mixing cases causes confusion in the SQL analytics endpoint and downstream tools. |
| **Separator** | Underscores (`_`) only. No spaces, hyphens, periods, or special characters. |
| **First character** | Must be a letter (not a number or underscore). |
| **Length** | Keep table names under 60 characters. Long names become unwieldy in SQL queries, notebook code, and Power BI field lists. |
| **No redundant words** | Do not include `table`, `tbl`, `delta`, or `data` in the name (e.g., use `sales_orders` not `sales_orders_table` or `sales_orders_data`). |

---

### 4.3 Table Naming — By Medallion Layer

Each layer has a distinct naming pattern that encodes the layer, the source provenance, and the business entity.

#### 4.3.1 Bronze Layer Tables

Bronze tables are raw replicas from source systems. Preserve the original entity name and always identify the source.

**Pattern:**
```
br_[source_system]_[entity_name]
```

| Segment | Description | Examples |
|---|---|---|
| `br_` | Bronze layer prefix | — |
| `source_system` | Abbreviated name of the originating system | `erp`, `crm`, `hris`, `sftp`, `api`, `sap`, `d365`, `sf` (Salesforce), `ga` (Google Analytics) |
| `entity_name` | The source table or entity, using the source system's own terminology where practical | `sales_orders`, `customers`, `gl_journal_entries`, `product_catalog` |

**Examples:**

| Table Name | Reading |
|---|---|
| `br_erp_sales_orders` | Bronze copy of the sales orders table from the ERP system |
| `br_crm_contacts` | Bronze copy of contacts from the CRM |
| `br_hris_employees` | Bronze copy of employee records from the HRIS |
| `br_sap_gl_journal_entries` | Bronze copy of general ledger entries from SAP |
| `br_sftp_vendor_invoices` | Bronze copy of vendor invoices received via SFTP |
| `br_api_exchange_rates` | Bronze copy of exchange rates from an external API |

**Additional bronze guidance:**

- If the source has schema-qualified tables (e.g., `dbo.SalesOrderHeader` in SQL Server), flatten it: `br_erp_sales_order_header`.
- If loading from files rather than tables, use the logical entity name the file represents, not the filename.
- Add metadata columns to every bronze table (not in the name, but in the schema): `_load_timestamp`, `_source_file`, `_pipeline_run_id`. These support auditability and reprocessing.

#### 4.3.2 Silver Layer Tables

Silver tables are cleansed, conformed, deduplicated, and type-cast. The source system prefix is dropped because silver represents an enterprise-conformed view.

**Pattern:**
```
sl_[entity_name]
```

**Examples:**

| Table Name | Reading |
|---|---|
| `sl_sales_orders` | Silver cleansed sales orders (potentially merged from multiple sources) |
| `sl_customers` | Silver conformed customer master |
| `sl_employees` | Silver conformed employee records |
| `sl_gl_journal_entries` | Silver conformed general ledger entries |
| `sl_product_catalog` | Silver conformed product catalog |

**When to retain source prefix in silver:** If you have the same entity arriving from multiple sources that have *not* yet been merged (e.g., customers from CRM and customers from ERP, pre-deduplication), you may temporarily retain the source prefix in silver: `sl_crm_customers`, `sl_erp_customers`. Once merged, the unified table drops the prefix: `sl_customers`.

**Silver with schemas (schema-enabled lakehouses):**

If using named schemas, you can optionally organize silver tables by business domain rather than relying solely on name prefixes:

```
schema: sales       → sl_orders, sl_order_lines
schema: finance     → sl_gl_entries, sl_ap_invoices
schema: hr          → sl_employees, sl_departments
```

#### 4.3.3 Gold Layer Tables

Gold tables are business-ready, analytics-optimized structures—typically star schemas (Kimball), wide denormalized tables, or domain-specific aggregations. Gold table names must communicate the *model role* (fact, dimension, bridge, aggregate) in addition to the business entity.

**Pattern:**
```
gd_[model_role]_[entity_name]
```

**Model Role Prefixes:**

| Combined Prefix | Role | Description |
|---|---|---|
| `gd_fct_` | Fact table | Transactional or event grain, contains measures and foreign keys |
| `gd_dim_` | Dimension table | Descriptive attributes used for slicing and filtering |
| `gd_bridge_` | Bridge table | Resolves many-to-many relationships between facts and dimensions |
| `gd_agg_` | Aggregate / Summary table | Pre-computed rollups for performance |
| `gd_snap_` | Snapshot table | Point-in-time snapshots (e.g., daily inventory levels, monthly balances) |
| `gd_map_` | Mapping / Crosswalk table | Maps codes or keys between systems |
| `gd_ref_` | Reference table | Slowly changing lookup data (e.g., country codes, currency codes) |

**Examples:**

| Table Name | Reading |
|---|---|
| `gd_fct_sales` | Gold fact table for sales transactions |
| `gd_fct_web_events` | Gold fact table for website clickstream events |
| `gd_fct_gl_journal` | Gold fact table for general ledger journal entries |
| `gd_dim_customer` | Gold customer dimension |
| `gd_dim_product` | Gold product dimension |
| `gd_dim_date` | Gold date/calendar dimension |
| `gd_dim_employee` | Gold employee dimension |
| `gd_dim_geography` | Gold geography dimension |
| `gd_bridge_customer_account` | Bridge resolving many-to-many between customer and account |
| `gd_agg_daily_sales_by_region` | Pre-aggregated daily sales by region |
| `gd_snap_monthly_inventory` | End-of-month inventory snapshot |
| `gd_ref_currency` | Currency code reference table |
| `gd_map_product_category` | Crosswalk mapping products to reporting categories |

**Gold with schemas:**

In a schema-enabled gold lakehouse, schemas can represent subject areas or data products:

```
schema: sales       → gd_fct_sales, gd_dim_customer, gd_dim_product, gd_dim_date
schema: finance     → gd_fct_gl_journal, gd_dim_cost_center, gd_dim_account
schema: hr          → gd_fct_headcount, gd_dim_employee, gd_dim_department
```

#### 4.3.4 Utility and Staging Tables

For tables that support pipeline operations but are not part of the bronze/silver/gold model:

| Prefix | Purpose | Example |
|---|---|---|
| `stg_` | Temporary staging tables used within a pipeline run | `stg_orders_dedup` |
| `tmp_` | Truly temporary tables that should be cleaned up after use | `tmp_load_validation_errors` |
| `meta_` | Metadata or control tables | `meta_pipeline_run_log`, `meta_watermark` |
| `audit_` | Audit and lineage tracking tables | `audit_row_counts`, `audit_data_quality_results` |

---

### 4.4 Column Naming

#### 4.4.1 General Column Rules

| Rule | Detail |
|---|---|
| **Case** | All lowercase with underscores: `order_date`, `customer_id`, `total_amount` |
| **No spaces** | Delta/Parquet will reject or silently drop columns with spaces. Fabric's Dataflow Gen2 can auto-fix unsupported characters to underscores, but this produces ugly names like `Amount__USD_`. Clean column names at the source or in transformation logic. |
| **No special characters** | Avoid parentheses, brackets, periods, dollar signs, percent signs, and any non-alphanumeric characters other than underscores. |
| **Be explicit** | Use full words where practical. `customer_id` is better than `cust_id`; `order_date` is better than `ord_dt`. Short abbreviations are acceptable for very common terms (see §4.4.3). |
| **No reserved words** | Avoid Spark SQL and T-SQL reserved words as column names (e.g., `date`, `timestamp`, `order`, `table`, `user`). Add a qualifier: `order_date`, `user_name`, `record_timestamp`. |

#### 4.4.2 Standard Column Suffixes

Consistent suffixes make data types and semantics immediately recognizable across every table in the environment.

| Suffix | Meaning | Data Type Expectation | Examples |
|---|---|---|---|
| `_id` | Surrogate or natural key | Integer or string | `customer_id`, `order_id`, `product_sk_id` |
| `_key` or `_sk` | Surrogate key (data warehouse) | Integer | `customer_sk`, `product_key` |
| `_bk` | Business key (natural key from source) | String or integer | `customer_bk`, `product_bk` |
| `_code` | Code or short identifier | String | `currency_code`, `country_code`, `status_code` |
| `_name` | Human-readable name | String | `customer_name`, `product_name`, `region_name` |
| `_desc` | Longer description | String | `product_desc`, `category_desc` |
| `_date` | Calendar date (no time) | Date | `order_date`, `ship_date`, `hire_date` |
| `_datetime` or `_ts` | Timestamp with time component | Timestamp | `created_datetime`, `modified_ts`, `event_ts` |
| `_amt` | Monetary amount | Decimal | `order_amt`, `discount_amt`, `tax_amt` |
| `_qty` | Quantity / count | Integer or decimal | `order_qty`, `stock_qty` |
| `_pct` | Percentage (stored as decimal, e.g., 0.15 = 15%) | Decimal | `discount_pct`, `margin_pct`, `tax_pct` |
| `_rate` | Rate or ratio | Decimal | `exchange_rate`, `interest_rate` |
| `_flag` | Boolean indicator | Boolean or tinyint | `is_active_flag`, `is_deleted_flag`, `has_promotion_flag` |
| `_num` | Non-key number / count | Integer | `line_num`, `version_num` |
| `_cat` | Category | String | `expense_cat`, `risk_cat` |
| `_type` | Type classifier | String | `address_type`, `payment_type` |
| `_url` | URL / hyperlink | String | `profile_url`, `document_url` |

#### 4.4.3 Approved Column Abbreviations

Keep this list short. Only abbreviate when the full word is excessively long and the abbreviation is universally understood.

| Abbreviation | Full Word |
|---|---|
| `id` | Identifier |
| `sk` | Surrogate Key |
| `bk` | Business Key |
| `fk` | Foreign Key |
| `desc` | Description |
| `amt` | Amount |
| `qty` | Quantity |
| `pct` | Percent |
| `ts` | Timestamp |
| `num` | Number |
| `cat` | Category |
| `addr` | Address |
| `org` | Organization |
| `dept` | Department |
| `mgr` | Manager |
| `curr` | Currency |
| `prev` | Previous |
| `avg` | Average |
| `min` | Minimum |
| `max` | Maximum |
| `ytd` | Year to Date |
| `mtd` | Month to Date |

#### 4.4.4 System and Metadata Columns

Every bronze table should include a consistent set of metadata columns for auditability. Use a leading underscore prefix (`_`) to visually separate metadata from business columns:

| Column Name | Purpose |
|---|---|
| `_load_ts` | Timestamp when the row was ingested into the lakehouse |
| `_source_file` | Name/path of the source file (for file-based ingestion) |
| `_source_system` | Identifier of the originating system |
| `_pipeline_run_id` | Unique ID of the pipeline run that loaded the row |
| `_is_deleted_flag` | Soft-delete indicator for CDC (Change Data Capture) patterns |
| `_valid_from_ts` | SCD Type 2 validity start |
| `_valid_to_ts` | SCD Type 2 validity end |
| `_is_current_flag` | SCD Type 2 current-record indicator |

#### 4.4.5 Column Naming by Layer

| Layer | Column Naming Approach |
|---|---|
| **Bronze** | Preserve original source column names wherever possible (lowercased, spaces replaced with underscores). Add `_` metadata columns. This preserves lineage traceability back to the source. |
| **Silver** | Rename to enterprise-standard names. Apply standard suffixes. Ensure consistency across entities from different sources (e.g., both ERP and CRM should use `customer_id`, not `cust_no` vs `client_id`). |
| **Gold** | Names should be business-friendly and report-ready. These are the names analysts and Power BI users will see in semantic models and reports. |

---

### 4.5 File Naming (Lakehouse `/Files` Section)

The `/Files` area is unmanaged storage—Fabric does not auto-discover or organize it for you. A disciplined naming and folder convention is essential to prevent it from becoming a dumping ground.

#### 4.5.1 Top-Level Folder Structure

Organize `/Files` by function first, then by source and entity:

```
/Files
├── landing/                          ← Raw files as received from sources
│   ├── [source_system]/
│   │   ├── [entity]/
│   │   │   ├── YYYY/MM/DD/           ← Date-partitioned subfolders
│   │   │   │   └── [files]
│   │   │   └── ...
│   │   └── ...
│   └── ...
├── archive/                          ← Processed files moved here for retention
│   ├── [source_system]/
│   │   ├── [entity]/
│   │   │   └── YYYY/MM/DD/
│   │   └── ...
│   └── ...
├── rejected/                         ← Files that failed validation
│   ├── [source_system]/
│   │   └── [entity]/
│   │       └── YYYY/MM/DD/
│   └── ...
├── reference/                        ← Static reference files (lookups, configs)
│   └── ...
├── temp/                             ← Short-lived working files (auto-purge policy)
│   └── ...
└── exports/                          ← Files generated for downstream consumption
    └── ...
```

**Why this structure?**

- **`landing/`** is the single entry point. Everything arrives here, gets processed, then moves.
- **`archive/`** preserves the raw file for audit and reprocessing without cluttering the landing zone.
- **`rejected/`** makes failed files visible instead of silently losing them.
- **`reference/`** holds slowly-changing lookup files that don't come through automated pipelines.
- **`temp/`** gives pipelines a scratch area. Implement an automated cleanup policy (e.g., delete files older than 7 days).
- **`exports/`** is for outbound files generated by notebooks or pipelines for downstream systems.

#### 4.5.2 File Naming Convention

**Pattern:**
```
[source_system]_[entity]_[YYYYMMDD]_[HHMMSS]_[sequence].[extension]
```

| Segment | Required? | Description |
|---|---|---|
| `source_system` | Yes | Same abbreviation used in table names (e.g., `erp`, `crm`, `sftp`) |
| `entity` | Yes | Logical entity the file represents |
| `YYYYMMDD` | Yes | Date the data pertains to (business date) or the extraction date |
| `HHMMSS` | Optional | Time of extraction, useful for intraday loads |
| `sequence` | Optional | A zero-padded sequence number (`001`, `002`) when multiple files arrive for the same entity and date |
| `extension` | Yes | File format: `.parquet`, `.csv`, `.json`, `.xlsx`, `.avro` |

**Examples:**

| File Name | Reading |
|---|---|
| `erp_sales_orders_20260415_001.parquet` | First sales orders extract from ERP for April 15, 2026 |
| `erp_sales_orders_20260415_002.parquet` | Second file for the same entity/date (e.g., large data split) |
| `crm_contacts_20260415_143022.csv` | CRM contacts extract at 14:30:22 on April 15, 2026 |
| `sftp_vendor_invoices_20260401.xlsx` | Vendor invoices received via SFTP for April 1, 2026 |
| `api_exchange_rates_20260415.json` | Daily exchange rate pull from API |

**Rules:**

- All lowercase, underscores only (no spaces or hyphens in filenames).
- Always include the date. Files without dates become impossible to manage at scale.
- Use the business date (the date the data represents), not necessarily the upload date, as the primary date component.
- For full-load (snapshot) files, use the snapshot date. For incremental files, use the date range or the extraction date.

#### 4.5.3 Date-Partitioned Folder Convention

For high-volume ingestion (daily or more frequent), organize files into Hive-style date folders:

**Standard Hive-style partitioning:**
```
/Files/landing/erp/sales_orders/year=2026/month=04/day=15/
    erp_sales_orders_20260415_001.parquet
    erp_sales_orders_20260415_002.parquet
```

**Simplified date partitioning (when Hive-style is not required):**
```
/Files/landing/erp/sales_orders/2026/04/15/
    erp_sales_orders_20260415_001.parquet
```

| Partition Style | When to Use |
|---|---|
| **Hive-style** (`year=YYYY/month=MM/day=DD`) | When the files will be queried directly via Spark or SQL analytics endpoint using `OPENROWSET`. Fabric's SQL engine can prune Hive-style partitions, improving query performance dramatically. |
| **Simplified** (`YYYY/MM/DD`) | When files are always processed through a pipeline/notebook and never queried directly. Simpler to navigate manually. |

> **Microsoft guidance:** Microsoft's medallion architecture documentation recommends using a partitioned folder structure wherever applicable, noting that it improves data manageability and query performance through partition pruning/elimination. For silver and gold layers, Microsoft recommends Liquid Clustering over traditional partitioning for optimized query performance.

#### 4.5.4 Archive and Retention Naming

When files are moved from `landing/` to `archive/` after processing, preserve the original path structure:

```
/Files/archive/erp/sales_orders/2026/04/15/
    erp_sales_orders_20260415_001.parquet         ← Same file, same name
```

For rejected files, append a rejection reason suffix before moving:

```
/Files/rejected/erp/sales_orders/2026/04/15/
    erp_sales_orders_20260415_001_schema_mismatch.parquet
    erp_sales_orders_20260415_002_empty_file.parquet
```

---

### 4.6 Table Partitioning Naming

When Delta tables are partitioned (especially in the bronze layer for high-frequency ingestion), use Hive-style partition column names that are lowercase, descriptive, and consistent:

| Partition Strategy | Column Name Convention | Example |
|---|---|---|
| By date | `year`, `month`, `day` | `/year=2026/month=04/day=15/` |
| By region | `region` | `/region=us_east/` |
| By source | `source_system` | `/source_system=erp/` |
| Combined | Multiple columns | `/year=2026/month=04/source_system=erp/` |

**Rules:**

- Partition column names must be lowercase with underscores.
- Do not use abbreviated date formats like `yr` or `mo`—spell out `year`, `month`, `day`.
- Keep partition cardinality manageable. Partitioning by a column with millions of unique values creates excessive small files and degrades performance.
- For silver and gold layers, prefer Liquid Clustering over traditional partitioning as recommended by Microsoft.

---

### 4.7 Lakehouse Schema Naming (Schema-Enabled Lakehouses)

Schema names follow the same character restrictions as table names (letters, numbers, underscores, must start with a letter).

**Pattern:**
```
[domain_or_subject_area]
```

**Examples:**

| Schema Name | Purpose |
|---|---|
| `dbo` | Default schema (auto-created) |
| `staging` | Temporary staging tables for pipeline processing |
| `sales` | Sales domain tables |
| `finance` | Finance domain tables |
| `hr` | Human resources domain tables |
| `marketing` | Marketing domain tables |
| `shared` | Cross-domain reference and utility tables |
| `audit` | Pipeline audit, quality, and control tables |

**Rules:**

- Schema names are singular nouns representing the domain (`sales` not `sales_tables`).
- Use the same domain names you use in your workspace and project naming for consistency.
- When referencing tables in code, always use fully qualified names: `schema_name.table_name` (e.g., `sales.gd_fct_sales`). This future-proofs your code for cross-lakehouse queries using four-part naming.

---

### 4.8 Comprehensive Lakehouse Example

Pulling together all the conventions from this section, here is a realistic view of what a well-named lakehouse looks like at each layer.

**Bronze Lakehouse: `sales_lh_100_bronze_raw`**

```
Tables/
├── dbo/
│   ├── br_erp_sales_orders
│   ├── br_erp_sales_order_lines
│   ├── br_erp_customers
│   ├── br_erp_products
│   ├── br_crm_contacts
│   ├── br_crm_opportunities
│   ├── br_api_exchange_rates
│   ├── meta_pipeline_run_log
│   └── meta_watermark

Files/
├── landing/
│   ├── erp/
│   │   ├── sales_orders/year=2026/month=04/day=15/
│   │   │   └── erp_sales_orders_20260415_001.parquet
│   │   ├── customers/year=2026/month=04/day=15/
│   │   │   └── erp_customers_20260415_001.parquet
│   │   └── products/year=2026/month=04/day=15/
│   │       └── erp_products_20260415_001.parquet
│   └── crm/
│       └── contacts/year=2026/month=04/day=15/
│           └── crm_contacts_20260415_143022.csv
├── archive/
│   └── erp/
│       └── sales_orders/year=2026/month=04/day=14/
│           └── erp_sales_orders_20260414_001.parquet
├── rejected/
│   └── crm/
│       └── contacts/year=2026/month=04/day=13/
│           └── crm_contacts_20260413_001_schema_mismatch.csv
└── reference/
    ├── currency_codes.csv
    └── country_codes.csv
```

**Silver Lakehouse: `sales_lh_200_silver_conformed`**

```
Tables/
├── sales/
│   ├── sl_sales_orders
│   ├── sl_sales_order_lines
│   └── sl_customers
├── marketing/
│   ├── sl_contacts
│   └── sl_opportunities
├── shared/
│   ├── sl_products
│   └── sl_exchange_rates
└── audit/
    ├── audit_data_quality_results
    └── audit_row_counts
```

**Gold Lakehouse: `sales_lh_300_gold_analytics`**

```
Tables/
├── sales/
│   ├── gd_fct_sales
│   ├── gd_fct_web_events
│   ├── gd_dim_customer
│   ├── gd_dim_product
│   ├── gd_dim_date
│   ├── gd_dim_geography
│   ├── gd_bridge_customer_account
│   ├── gd_agg_daily_sales_by_region
│   ├── gd_snap_monthly_inventory
│   └── gd_ref_currency
└── shared/
    └── gd_dim_date
```

---

## 5. Abbreviation Registry

Maintain a single source of truth for all approved abbreviations. Below is the starter registry. Extend it as your environment grows, but **never allow duplicates or conflicts**.

### 5.1 Fabric Artifact Type Abbreviations

| Artifact | Abbreviation |
|---|---|
| Lakehouse | `lh` |
| Warehouse | `wh` |
| Pipeline | `pl` |
| Dataflow Gen2 | `dfl` |
| Notebook | `nb` |
| Semantic Model | `sm` |
| Report | `rpt` |
| Dashboard | `dsh` |
| Spark Job Definition | `sj` |
| ML Experiment | `exp` |
| ML Model | `mdl` |
| KQL Database | `kdb` |
| KQL Queryset | `kqs` |
| Eventstream | `es` |
| Data Activator / Reflex | `rfx` |
| Mirrored Database | `mir` |
| Environment | `env` |

### 5.2 Fabric Experience Group Abbreviations (Optional Segment)

| Experience | Abbreviation |
|---|---|
| Power BI | `pbi` |
| Data Factory | `df` |
| Data Engineering (Synapse) | `de` |
| Data Science (Synapse) | `ds` |
| Data Warehouse (Synapse) | `dw` |
| Real-Time Intelligence | `rti` |
| Data Activator | `da` |

### 5.3 Medallion Layer Abbreviations

| Layer | Full Name | Index |
|---|---|---|
| `bronze` | Raw / Landing | `100` |
| `silver` | Cleansed / Conformed | `200` |
| `gold` | Business / Curated | `300` |

### 5.4 Common Domain / Project Abbreviations

| Domain | Abbreviation |
|---|---|
| Finance | `fin` |
| Human Resources | `hr` |
| Sales | `sales` |
| Marketing | `mktg` |
| Supply Chain | `supply` |
| Customer | `cust` |
| Operations | `ops` |
| Enterprise (shared/cross-functional) | `ent` |

---

## 6. Deployment Pipelines and Environment Strategy

Microsoft Fabric's Deployment Pipelines feature auto-pairs artifacts between stages by name. Therefore:

- **Do NOT include the environment (dev/test/prod) in artifact names.** If you do, auto-pairing breaks and you must manually reconfigure mappings after every deployment.
- **DO include the environment in workspace names** (via the `[Dev]`, `[Test]` suffix per §3.1). This cleanly separates environments without creating naming divergence inside the workspace.

This recommendation aligns with the XTIVIA framework, which deliberately omitted environment from artifact names to preserve Deployment Pipeline compatibility.

---

## 7. Governance and Enforcement

### 7.1 Certification Criteria

Consider making adherence to this naming standard a prerequisite for artifact certification (the Fabric "Endorsed" or "Certified" badge). Advancing Analytics recommends this as a practical enforcement mechanism.

### 7.2 Workspace Creation Controls

Microsoft recommends using the Fabric Admin Portal to control who can create workspaces (via the *Create workspaces* tenant setting). Restrict workspace creation to a trained set of users (e.g., a "Fabric Workspace Creators" security group) who are familiar with this standard.

### 7.3 Onboarding Checklist

Every new team member should receive this standard document and confirm understanding before creating any artifacts. A simple validation checklist:

- Does the name use all lowercase with underscores?
- Does it contain a recognized artifact type abbreviation?
- Can you identify the project, the data layer, and the purpose from the name alone?
- Is it under 80 characters?
- Are all abbreviations from the approved registry?

### 7.4 Exception Process

If a scenario requires deviating from this standard (e.g., a vendor-mandated name), document the exception, get approval from the data governance lead, and add a description/tag on the artifact explaining the deviation.

---

## 8. Quick-Reference Examples

A realistic Fabric workspace listing for a Sales Analytics project, sorted alphabetically:

```
Workspace: Sales-Analytics

sales_dfl_100_bronze_ingest_crm_contacts
sales_dfl_200_silver_transform_customer_dim
sales_exp_churn_prediction
sales_lh_100_bronze_erp_orders
sales_lh_100_bronze_crm_contacts
sales_lh_200_silver_orders
sales_lh_200_silver_customers
sales_lh_300_gold_sales_mart
sales_mdl_churn_prediction_xgb
sales_nb_100_bronze_raw_validation
sales_nb_200_silver_transform_orders
sales_nb_200_silver_transform_customers
sales_nb_300_gold_build_sales_mart
sales_nb_900_utility_schema_migration
sales_pl_000_orchestrator_daily_refresh
sales_pl_100_bronze_ingest_erp_orders
sales_pl_200_silver_transform_orders
sales_pl_300_gold_load_sales_mart
sales_rpt_executive_dashboard
sales_rpt_regional_sales_breakdown
sales_sm_sales_performance
sales_wh_gold_sales_analytics
```

Notice how the alphabetical sort naturally groups artifacts by type and then by pipeline stage, making the entire workspace self-documenting.

---

## 9. Sources and References

1. **Microsoft — Define Your Naming Convention (Cloud Adoption Framework):** [learn.microsoft.com/...resource-naming](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)
2. **Microsoft — Abbreviation Recommendations for Azure Resources:** [learn.microsoft.com/...resource-abbreviations](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations)
3. **Microsoft — Workspace Tenant-Level Planning:** [learn.microsoft.com/...workspaces-tenant-level-planning](https://learn.microsoft.com/en-us/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning)
4. **Microsoft — Best Practices for Domains:** [learn.microsoft.com/...domains-best-practices](https://learn.microsoft.com/en-us/fabric/governance/domains-best-practices)
5. **Microsoft — Implement Medallion Lakehouse Architecture in Fabric:** [learn.microsoft.com/...onelake-medallion-lakehouse-architecture](https://learn.microsoft.com/en-us/fabric/onelake/onelake-medallion-lakehouse-architecture)
6. **Microsoft — Lakehouse and Delta Lake Tables:** [learn.microsoft.com/...lakehouse-and-delta-tables](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-and-delta-tables)
7. **Microsoft — Lakehouse Schemas:** [learn.microsoft.com/...lakehouse-schemas](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-schemas)
8. **Microsoft — Navigate the Fabric Lakehouse Explorer:** [learn.microsoft.com/...navigate-lakehouse-explorer](https://learn.microsoft.com/en-us/fabric/data-engineering/navigate-lakehouse-explorer)
9. **Microsoft — Load Data to Lakehouse Using Partition:** [learn.microsoft.com/...tutorial-lakehouse-partition](https://learn.microsoft.com/en-us/fabric/data-factory/tutorial-lakehouse-partition)
10. **Advancing Analytics — "What's in a Name? Naming Your Fabric Artifacts"** (Johnny Winter, 2023): [advancinganalytics.co.uk](https://www.advancinganalytics.co.uk/blog/2023/8/16/whats-in-a-name-naming-your-fabric-artifacts)
11. **XTIVIA — Microsoft Fabric Resources Naming Conventions: Why and How?** (Pavan Malpani, 2024): [xtivia.com](https://www.xtivia.com/blog/microsoft-fabric-resource-naming-conventions/)
12. **Fabric.guru — Thoughts on Spaces in Workspace and Column Names:** [fabric.guru](https://fabric.guru/thoughts-on-spaces-in-workspace-and-column-names-in-microsoft-fabric)
13. **That Fabric Guy — Medallion Architecture for Microsoft Fabric, a Theoretical Guide:** [thatfabricguy.com](https://thatfabricguy.com/medallion-architecture-microsoft-fabric-theory-guide/)
14. **Data Crafters — The Curious Case of Column Names in Microsoft Fabric Lakehouse:** [datacrafters.io](https://datacrafters.io/the-curious-case-of-column-names-in-microsoft-fabric-lakehouse/)

---

*This is a living document. Review annually or whenever Microsoft publishes official Fabric naming guidance.*
