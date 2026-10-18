# Atlikon - SportsBar End-to-End Lakehouse Pipeline

An enterprise data lakehouse platform designed to ingest, process, curate, and serve transactional sales data from child operations (**SportsBar**) to the enterprise analytics plane (**Atlikon**). 

The platform implements a **Medallion Architecture** using **Databricks Lakeflow Jobs**, governed centrally under **Databricks Unity Catalog**, and exposes business-ready star-schema marts to **BI Dashboards** and **Databricks Genie Spaces**.

---

## Architecture Overview

The system streams and batches operational data from transactional OLTP sources into Amazon S3, transformations through a Medallion pattern, and provisions consolidated multi-tenant Gold models for parent enterprise reporting.


<img width="1388" height="604" alt="tech-architecture" src="https://github.com/user-attachments/assets/d0d3095d-69cf-43fc-a9bf-9a85b67dd63f" />


### Components

* **Source Layer (OLTP):** SportsBar transactional database emitting operational events.
* **Storage Layer (AWS S3):** Object store managing incoming raw files and point-in-time archives.
* **Processing & Governance:** Databricks compute governed by **Unity Catalog** for role-based access control (RBAC), data discovery, and lineage tracking.
* **Orchestration:** **Databricks Lakeflow Jobs** automating extraction, transformation, data quality checks, and scheduled aggregation runs.
* **Serving Layer:**
  * **Databricks Dashboards:** Executive reporting and operational KPIs.
  * **Databricks Genie:** Conversational, natural-language ad-hoc query space powered by the underlying Gold semantic model.

---

## Data Model (Gold Layer)

The analytical store implements a **Star Schema** centered around order transactions, supporting product performance, pricing trends, and customer geographic analytics.

<img width="1067" height="696" alt="data-model" src="https://github.com/user-attachments/assets/b86be575-c9b0-4121-83e7-a3b94b78c8f0" />


---

## Medallion Data Flow

1. **Bronze Layer:**
   * Raw ingestion from `s3://<bucket>/raw_data/` and historical recovery from `s3://<bucket>/archive/`.
   * Preserves raw JSON/CSV schemas with ingestion timestamps and lineage metadata.
2. **Silver Layer:**
   * Schema enforcement, deduplication, datatype casting, null handling, and surrogate key generation.
   * Cleansed transactional and entity data written as Delta tables.
3. **Gold Layer:**
   * Business-level aggregations and dimensional modeling (`fact_orders`, `dim_*`).
   * **Parent ORG Sync:** Curated views and materialized tables replicated to `atlikon_catalog.gold_analytics` for executive multi-subsidiary cross-analysis.

---

## Deployment & Setup

### Prerequisites

* AWS Account with an established S3 bucket (`raw-data` and `archive` prefixes).
* Databricks Workspace enabled with **Unity Catalog**.
* Databricks CLI configured (`databricks configure`).
* Python 3.10+ & SQLFluff for linting.

### 1. Catalog & Permissions Setup

Ensure catalogs are registered in Unity Catalog:

```sql
CREATE CATALOG IF NOT EXISTS sportsbar_catalog;
CREATE CATALOG IF NOT EXISTS atlikon_catalog;

CREATE SCHEMA IF NOT EXISTS sportsbar_catalog.bronze;
CREATE SCHEMA IF NOT EXISTS sportsbar_catalog.silver;
CREATE SCHEMA IF NOT EXISTS sportsbar_catalog.gold;
CREATE SCHEMA IF NOT EXISTS atlikon_catalog.gold_analytics;

```

### 2. Configure Lakeflow Job

Deploy the orchestration pipeline via Databricks Asset Bundles (DABs) or CLI:

```bash
databricks bundle deploy --target prod
databricks bundle run lakeflow_pipeline

```

### 3. Verification & Serving Layer Setup

1. **Verify Unity Catalog Entities:** Confirm tables exist under `sportsbar_catalog.gold` and are queryable.
2. **Configure Genie Space:**
* Navigate to **Databricks SQL > Genie**.
* Create a new space: `SportsBar Order Performance`.
* Attach `sportsbar_catalog.gold` tables and provide the column descriptions from the data dictionary.


3. **Publish Dashboard:** Import `dashboards/sportsbar_executive_kpis.lvdash.json` into the Databricks SQL workspace.

---

## Monitoring & Data Governance

* **Unity Catalog Lineage:** Visual lineage tracking is active from `s3://` ingestion paths down to gold metric views.
* **Lakeflow Monitoring:** Set email/Slack alerts on job failure or schema drift within the Databricks Jobs UI.
* **Data Quality Expectations:** Managed through Delta Live Tables / Lakeflow pipeline expectations to drop or quarantine invalid records (e.g., negative `order_qty` or null `product_id`).
