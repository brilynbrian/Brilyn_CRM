# Brilyn CRM Modernization Implementation

This repository currently contains the implementation blueprint for modernizing a legacy SQL + Visual Studio CRM into a React, Dataverse, Azure SQL, Azure Data Factory, Power BI, and Data Warehouse architecture.

## 1) Program Goal

Migrate from:
- Legacy UI + Business + SQL layers
- SQL-heavy logic in tables, views, and stored procedures

To:
- React UI (developed in VS Code)
- Hybrid Dataverse + Azure SQL operational platform
- Azure Data Factory integration (near-real-time + orchestrated batch)
- Payment platform integration (XERO, Stripe, eWAY tokenized flows)
- Power BI on top of a dedicated data warehouse and semantic model

---

## 2) Discovery-First Workstream (Immediate)

### 2.1 Legacy Asset Inventory

Capture and catalogue:
- SQL tables, views, stored procedures, triggers, SQL Agent jobs
- SSIS packages and integration jobs
- Reporting objects and downstream dependencies
- Payment-related data paths and reconciliation rules

For each object, capture:
- Domain (Customer, Sales, Billing, Service, Reporting, Integration)
- Owner (team/person/system)
- Update frequency (real-time, hourly, daily, ad-hoc)
- Criticality (P0/P1/P2/P3)
- Type (operational, analytical, integration-only)
- Target disposition (keep, refactor, retire, replace)

### 2.2 Stored Procedure Classification

Tag each procedure as:
- CRUD/UI-coupled
- Business-rule heavy
- Batch/reconciliation
- Reporting extract
- Integration bridge

Migration decision guidance:
- CRUD/UI-coupled -> move to API/service layer
- Business-rule heavy -> retain temporarily, then progressively refactor
- Batch/reconciliation -> retain if efficient, optimize later
- Reporting extract -> replace with warehouse models
- Integration bridge -> transition to ADF/event-driven patterns

---

## 3) Target-State Platform Split

### 3.1 Dataverse (preferred for CRM-native entities)
- Accounts, contacts, activities, opportunities, cases, workflow-driven entities
- Security roles, auditing, business process flows, Power Platform integration

### 3.2 Azure SQL (preferred for heavy transactional/relational loads)
- Payments, ledger, reconciliation, high-volume histories
- Complex relational workloads and high-throughput integration tables

### 3.3 Data Warehouse (analytics at scale)
- Historical fact storage, large analytical datasets, long-term archives
- Star-schema model for Power BI consumption

Rule: define a single system-of-record per entity to avoid dual-master conflicts.

---

## 4) Data Placement Decision Matrix

Use this matrix per table/view/proc dependency:

| Criterion | Dataverse | Azure SQL | Data Warehouse |
|---|---|---|---|
| CRM workflow and role-based behavior | Strong fit | Limited | No |
| Very high transaction throughput | Limited | Strong fit | No |
| Deep relational joins and procedural logic | Limited | Strong fit | No |
| Historical analytics and trend reporting | Limited | Limited | Strong fit |
| Near-real-time operational integration | Moderate | Strong fit | Moderate |
| Large cold data retention | Limited | Moderate | Strong fit |

---

## 5) Integration Architecture (ADF + Events)

### 5.1 ADF Responsibilities
- Orchestrated ingestion and transformation pipelines
- Near-real-time micro-batches where acceptable latency exists
- Data movement between operational stores and warehouse

### 5.2 Event-Driven Responsibilities
- Low-latency business events (payment status, customer updates, critical notifications)
- Decoupled propagation to downstream consumers

### 5.3 Integration Controls
- Canonical contracts for Customer, Invoice, Payment, Reference Data
- Idempotency, replay handling, dead-letter/error handling
- End-to-end lineage and observability

---

## 6) Payment Platform Integration (XERO, Stripe, eWAY)

Implementation principles:
- Isolate provider-specific logic behind a payment service layer
- Keep tokenized references only in CRM-facing stores
- Avoid storing PCI-sensitive card data in CRM databases
- Standardize events for authorize/capture/refund/failure/reconciliation
- Maintain auditable provider transaction trails

---

## 7) Reporting and Analytics

### 7.1 Power BI Source of Truth
- Use warehouse/semantic models for analytical reporting
- Avoid direct heavy reporting queries on operational CRM databases

### 7.2 Core Analytical Models
- Facts: sales, payments, service interactions, pipeline progression
- Dimensions: customer, product/service, channel, time, territory, agent

### 7.3 Reporting Separation
- Operational dashboards: low-latency operational KPIs
- Analytical dashboards: trend, cohort, and historical performance

---

## 8) Performance and Reliability Strategy

Priorities during migration:
- Baseline top slow queries/procedures and lock contention patterns
- Introduce targeted indexing and partitioning for large tables
- Remove cursor-heavy and multi-purpose view anti-patterns
- Introduce archival/retention policies to keep hot operational data lean
- Shift UI-coupled SQL logic to service APIs to reduce schema fragility

---

## 9) Agile Delivery Model

### 9.1 Delivery Strategy
- Deliver by business domain (vertical slices), not by technical layer only
- Start with: customer/account + payments + integration + reporting slice
- Use incremental migration with compatibility interfaces

### 9.2 Engineering Controls
- Schema and pipeline versioning
- Environment promotion gates
- Automated test suites for API, data contracts, and reconciliation
- Deployment pipelines for React, services, ADF, and BI assets

### 9.3 Non-Functional Done Criteria
- Performance baseline met
- Security and compliance checks passed
- Reconciliation and audit controls verified
- Rollback and observability in place

---

## 10) Required Discovery Outputs

Produce and maintain:
1. Current-state data model map
2. Procedure/view dependency matrix
3. Table migration decision matrix (Dataverse vs Azure SQL vs Warehouse)
4. Integration architecture and real-time event map
5. Reporting source-of-truth map
6. Performance risk register
7. Domain-based phased migration roadmap

This README is the baseline implementation guide for those outputs.
