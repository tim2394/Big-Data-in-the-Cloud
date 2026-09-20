# Pricing Estimate

## 1. Purpose

This document summarizes the cost assumptions for the Team 9 Big Data in the Cloud exam solution.

The exam case requires the team to design and deploy a cloud solution while keeping total Azure consumption within the team budget of **CHF 250**. The architecture is therefore designed to use small, pay-as-you-go resources and to run compute-intensive services only when required.

## 2. Costed Architecture

The estimate covers the following Azure services:

| Service | Role in the solution | Cost approach |
|---|---|---|
| Azure Blob Storage | Stores the raw food-price CSV and future input files | Small capacity, low transaction volume |
| Azure Data Factory | Orchestrates ingestion and processing steps | Low number of activity runs and execution hours |
| Azure SQL Database | Stores structured food-price data and analysis outputs | Small single database, Basic tier |
| Azure Databricks | Runs Python forecasting and correlation analysis | Small compute, pay-as-you-go, limited runtime |
| Azure Logic Apps | Sends operational notifications, e.g. pipeline failure | Consumption-based |
| Azure Monitor | Monitoring and alerting | Minimal logging/alert usage |
| Power BI Desktop | Dashboard and exam presentation | Local desktop tool; not included in Azure consumption |

## 3. Pricing Assumptions

### 3.1 Azure Blob Storage

The project uses a single static CSV as the initial source dataset. Storage requirements are therefore small.

**Assumptions**
- Region: Switzerland North
- Performance: Standard
- Redundancy: LRS
- Access tier: Hot
- Estimated capacity: approximately **5-10 GB**
- Read operations: low
- Write operations: low
- List/Create Container operations: low
- Data retrieval: only a few GB

The transaction assumptions are intentionally conservative. The calculator prices many storage actions in blocks of 10,000 operations, while this project is expected to use far fewer operations.

### 3.2 Azure SQL Database

Azure SQL Database is used as the structured relational storage layer for cleaned food-price data and analytical outputs.

**Current calculator configuration**
- Region: Switzerland North
- Type: Single Database
- Purchase model: DTU
- Service tier: Basic
- Performance level: 5 DTUs
- Included storage: 2 GB
- Number of databases: 1
- Backup redundancy: LRS
- Estimated monthly database cost: approximately **USD 5.39**

Long-term retention is not required for the exam prototype.

### 3.3 Azure Data Factory

Azure Data Factory is used to orchestrate the workflow, especially the movement of the source data from Blob Storage to Azure SQL and the optional triggering of Databricks processing.

**Assumptions**
- Region: Switzerland North
- Type: Azure Data Factory V2
- Service type: Data Pipeline
- Activity runs: approximately **100 activity runs/month**
- Data movement execution: approximately **5 hours/month**
- Pipeline activity execution: low
- External activity execution: approximately **5 hours/month** if Databricks is triggered from ADF
- Self-hosted Integration Runtime: not used
- Mapping Data Flow compute: not used

Mapping Data Flow compute is intentionally excluded because the main analytical transformations are performed in Databricks.

### 3.4 Azure Databricks

Databricks is used for Python-based forecasting, correlation analysis, and other analytical processing.

**Assumptions**
- Region: Switzerland North
- Workload: All-Purpose Compute during development
- Tier: Standard
- Small VM instance
- Pay-as-you-go
- No reservation or savings plan
- Estimated runtime: approximately **20 hours/month**
- Compute is stopped when not in use
- Auto-termination should be enabled in the deployed solution

Databricks pricing contains two main components:
1. the Azure VM compute cost, and
2. the Databricks Unit (DBU) platform charge.

The solution avoids a 24/7 cluster. Databricks runs only when forecasts or analytical outputs must be recalculated.

### 3.5 Azure Logic Apps

Logic Apps is used for lightweight operational notifications, such as sending an email if an ADF pipeline fails.

**Assumptions**
- Consumption model
- Low number of executions
- Only a few actions per execution
- No dedicated Standard instance

A dedicated Standard Logic Apps plan was rejected because it would add a large fixed monthly cost that is not justified by this project.

### 3.6 Azure Monitor

Azure Monitor is used for operational monitoring and alerting.

**Assumptions**
- Minimal log ingestion
- Small number of alerts
- No production-scale monitoring workload

## 4. Current Monthly Estimate

After replacing unrealistic 24/7 defaults with usage assumptions that match the exam scenario, the current Azure Pricing Calculator estimate is approximately:

> **USD 19 per month**

This value is an estimate and should be updated if the final architecture or usage assumptions change.

## 5. Cost-Optimization Decisions

The main cost-control decisions are:

- use small storage capacity because the source dataset is limited in size;
- use Azure SQL Basic rather than a larger production tier;
- keep ADF execution volumes low because the source is initially static;
- avoid ADF Mapping Data Flow compute where Databricks already performs analysis;
- run Databricks only when required rather than 24/7;
- enable Databricks auto-termination;
- use Logic Apps Consumption rather than a dedicated Standard plan;
- use Power BI Desktop for the exam presentation, avoiding unnecessary cloud BI licensing for the prototype.

## 6. Production Consideration

The exam solution is a prototype. A production deployment for a government customer could require higher availability, larger database capacity, more frequent refreshes, more monitoring, and a shared BI platform rather than Power BI Desktop.

Those production requirements would increase the recurring monthly cost and should be estimated separately if requested.

## 7. Cost Rationale

The architecture follows a cloud cost principle used throughout the project:

> Keep persistent storage inexpensive and run expensive compute only when required.

For example, the forecast results can remain stored in Azure SQL while Databricks is shut down. Government users can therefore access the latest calculated results without requiring the Databricks compute environment to remain active continuously.
