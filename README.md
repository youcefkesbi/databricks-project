# Atlikon / SportsBar End-to-End Lakehouse Pipeline

An enterprise data lakehouse platform simulating a Fast-Moving Consumer Goods (FMCG) acquisition scenario. **Atlikon**, a manufacturer of athletic and sports equipment, acquired **SportsBar**, a startup in the energy bar and athletic nutrition space. 

This pipeline ingests, harmonizes, and consolidates operational data from both entities into a single unified lakehouse using the **Medallion Architecture**, **Databricks Lakeflow Jobs**, **Databricks Unity Catalog**, **Amazon S3**, and **Apache Spark**, serving analytical data models to **BI Dashboards** and **Databricks Genie**.

---

## Architecture Overview

Operational OLTP sales data from the acquired child entity (SportsBar) is staged into Amazon S3, ingested via Databricks Lakeflow Jobs, processed through Bronze and Silver layers, and aggregated into a Gold analytics layer aligned with the parent organization (Atlikon)[cite: 2].


<img width="1388" height="604" alt="tech-architecture" src="https://github.com/user-attachments/assets/d0d3095d-69cf-43fc-a9bf-9a85b67dd63f" />


### Components

* **Source Layer (OLTP):** SportsBar transactional database emitting nutritional product sales and order streams[cite: 2].
* **Storage Layer (AWS S3):** Object store managing incoming raw files and point-in-time archives (`s3://.../raw_data/` and `s3://.../archive/`)[cite: 2].
* **Processing & Governance:** PySpark on Databricks governed centrally via **Unity Catalog** for access control and end-to-end lineage tracking[cite: 2].
* **Orchestration:** **Databricks Lakeflow Jobs** automating extraction, bronze-to-gold transformations, and data quality validations[cite: 2].
* **Serving Layer:**
  * **BI Dashboards:** Reporting on post-acquisition revenue, customer adoption, and order volumes[cite: 2].
  * **Databricks Genie:** Conversational natural-language interface allowing stakeholders to query combined metrics[cite: 2].

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
