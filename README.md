**Lakehouse Platform with Medallion Architecture**

This repository contains the design and implementation of a secure, scalable, and cost-efficient Lakehouse platform on Azure using the Medallion (Bronze-Silver-Gold) architecture pattern. The platform ingests data from various source systems and prepares analysis-ready datasets for data consumers.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Design Goals](#design-goals)
3. [Platform Components](#platform-components)
4. [Environment Setup](#environment-setup)
5. [Data Ingestion](#data-ingestion)
6. [Medallion Layers](#medallion-layers)

   * Bronze Zone
   * Silver Zone
   * Gold Zone
7. [Analysis Datasets](#analysis-datasets)
8. [Cost and Cluster Management](#cost-and-cluster-management)
9. [Data Governance](#data-governance)
10. [CI/CD and Testing](#cicd-and-testing)
11. [Contributing](#contributing)
12. [License](#license)

---

## Architecture Overview

A high-level view of the Lakehouse platform:

1. **Data Sources**

   * Azure SQL (batch via ADF)
   * Kafka Topics (streaming via Kafka Connect)
2. **Ingestion**

   * Azure Data Factory pipelines (batch & continuous)
   * Kafka Connect → ADLS Gen2
3. **Storage**

   * ADLS Gen2 with hierarchical containers:

     * `/sbit-metastore-root`
     * `/sbit-managed-dev/{bronze_db,silver_db,gold_db}`
     * `/sbit-unmanaged-dev/{data_zone,checkpoint_zone}`
4. **Processing**

   * Databricks workspaces: Dev, Test, Prod
   * Spark applications for Medallion transformations
5. **Serving**

   * Delta tables in managed catalogs
   * BI and analytics consumption

---

## Design Goals

1. **Environment Isolation**: Dev, Test, and Prod workspaces and storage.
2. **Decoupled Ingestion & Processing**: Separate pipelines for data landing and transformation.
3. **Hybrid Workflows**: Support batch (ADF) and streaming (Kafka Connect + Structured Streaming).
4. **Automated Testing**: Integration tests triggered in CI for data quality.
5. **Automated Deployment**: Infrastructure-as-Code and CI/CD for Test & Prod deployments.

---

## Platform Components

| Component                    | Purpose                                 |
| ---------------------------- | --------------------------------------- |
| **ADLS Gen2**                | Storage layer (Delta Lake compatible)   |
| **Azure Data Factory (ADF)** | Batch/continuous ingestion              |
| **Kafka Connect**            | Stream ingestion to ADLS Gen2           |
| **Databricks**               | Spark compute, Unity Catalog, notebooks |
| **Unity Catalog**            | Data governance & fine-grained access   |
| **Azure SQL**                | Source system for user and session data |

---

## Environment Setup

1. **Provision Azure Resources**

   * Create Resource Group
   * Create ADLS Gen2 Storage Account
   * Create Storage Containers: `sbit-managed-dev`, `sbit-unmanaged-dev`
2. **Databricks Workspace**

   * Create Dev/Test/Prod workspaces (Premium tier)
   * Attach Unity Catalog metastore:

     * Metastore root: `/sbit-metastore-root`
     * Managed catalog location: `/sbit-managed-dev`
     * Unmanaged location: `/sbit-unmanaged-dev/data_zone`
     * Checkpoint location: `/sbit-unmanaged-dev/checkpoint_zone`
3. **Networking & Security**

   * Configure storage access via Azure Databricks access connector
   * Set up VNet, NSG, and private endpoints as needed

---

## Data Ingestion

### Azure SQL → ADLS Gen2 (Batch)

* **Entities**:

  1. Registration
  2. Login/Logout
  3. User Profile (CDC)
  4. Workout Session
* **Flow**: ADF pipeline (`/reg-user`, `/gym-log`) → JSON files in `/data_zone`

### Kafka Topics → ADLS Gen2 (Streaming)

* **Topics**:

  1. BPM Stream
  2. Workout Session Updates
* **Flow**: Kafka Connect ADLS Sink → JSON in `/data_zone`

---

## Medallion Layers

### Bronze Zone

* Raw ingested JSON files copied into Delta tables without transformation.
* **Tables**:

  * `bronze.registration`
  * `bronze.gymlog`
  * `bronze.user_profile_cdc`
  * `bronze.bpm_stream`
  * `bronze.workout_session`

### Silver Zone

* Cleaned and conformed data with schema enforcement.
* **Transformations**:

  * Deduplication & schema casting
  * Join CDC changes to user profile
  * Extract heart rate events
  * Normalize workout sessions
* **Tables**:

  * `silver.users`
  * `silver.gym_logs`
  * `silver.user_bins`
  * `silver.workout_bpm`

### Gold Zone

* Aggregated, business-ready tables for analytics.
* **Datasets**:

  * `gold.workout_bpm_summary`
  * `gold.gym_summary`

---

## Analysis Datasets

1. **Workout BPM Summary**: Average/min/max BPM per user per session.
2. **Gym Summary**: Daily/weekly/monthly user attendance and session counts.

---

## Cost and Cluster Management

* **Workspace Management**: Separate Dev/Test/Prod workspaces
* **Cluster Policies**:

  1. Restrict node type and size
  2. Limit clusters per user
  3. Cap cluster cost
  4. Enforce autoscaling limits
  5. Pre-installed libraries & centralized log locations

---

## Data Governance

* **Unity Catalog**: Role-based access controls on catalogs, schemas, and tables.
* **Audit**: Table-level lineage and audit logs via Unity Catalog.

---

## CI/CD and Testing

1. **Infrastructure as Code**: Terraform scripts for Azure & Databricks resources
2. **CI Pipeline**: GitHub Actions / Azure DevOps

   * Deploy to Test
   * Run integration tests (data quality checks)
   * Approve & Deploy to Prod
3. **Testing**:

   * Schema validation
   * Row counts & null checks
   * Sample data assertions

---

## Contributing

Welcome contributions! Please fork the repo, create a feature branch, and submit a pull request.

---

## License

MIT License
