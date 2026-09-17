# Project brief: Capture Capacity Metrics Data

## Purpose

Build an automated pipeline that captures data from the Microsoft Fabric Capacity Metrics App and stores it in a Fabric lakehouse for retention, analysis, and operational reporting.

The Capacity Metrics App is useful for day-to-day monitoring, but its data is not a durable analytics store. This project turns those capacity signals into governed lakehouse data so teams can trend utilization, investigate throttling, monitor workload behavior, and connect capacity health to business or operational events.

## Problem to solve

Fabric administrators and engineering teams need a reliable way to preserve capacity metrics beyond the app experience. Today, capacity observations can be hard to retain, join with other datasets, or use in custom alerting and reporting. The project should remove that manual gap by landing the data automatically in OneLake.

## Goals

1. Capture Fabric Capacity Metrics App data on a recurring schedule.
2. Store raw and curated capacity data in a Fabric lakehouse.
3. Preserve enough history to support trend analysis, diagnostics, and chargeback/showback use cases.
4. Provide a clean schema for downstream notebooks, SQL endpoint queries, semantic models, and reports.
5. Build basic monitoring so failed captures are visible quickly.

## Out of scope for the first release

1. Replacing the Capacity Metrics App user experience.
2. Automated capacity scaling or pause/resume actions.
3. Full FinOps allocation logic by department, project, or application.
4. Cross-tenant collection.

## Proposed approach

Use a Fabric-native ingestion pattern that extracts the relevant metrics from the Capacity Metrics App dataset or supported admin APIs, lands the data in a Bronze layer, validates and normalizes it into Silver tables, then publishes Gold tables for reporting.

The preferred implementation should use Fabric Pipelines for orchestration, notebooks or Dataflows Gen2 for transformation, and a lakehouse as the system of record. If direct app dataset extraction is limited, the implementation should fall back to the supported Fabric admin and capacity APIs where they provide equivalent metrics.

## Target architecture

1. Source: Fabric Capacity Metrics App dataset and/or supported Fabric admin capacity metrics APIs.
2. Ingestion: Scheduled Fabric Pipeline that runs on an agreed cadence, such as hourly or daily.
3. Bronze: Raw extracts stored with capture timestamp, source metadata, and no destructive transformations.
4. Silver: Cleaned Delta tables with normalized capacity, workspace, item, operation, user, time, and metric fields.
5. Gold: Aggregated tables for utilization trends, throttling patterns, workload breakdowns, capacity pressure, and anomaly review.
6. Consumption: Semantic model, Power BI report, SQL endpoint queries, and optional Activator alerts.

## Initial data model

| Layer | Table | Purpose |
| --- | --- | --- |
| Bronze | `capacity_metrics_raw` | Raw source payloads with ingestion metadata. |
| Silver | `capacity_utilization` | Time-series CU usage, smoothing, peaks, and averages. |
| Silver | `capacity_throttling_events` | Interactive and background throttling signals. |
| Silver | `capacity_workload_activity` | Workload-level usage by item, workspace, and operation. |
| Silver | `capacity_items` | Item metadata for reports, semantic models, warehouses, lakehouses, notebooks, pipelines, and other Fabric items. |
| Silver | `capacity_workspaces` | Workspace metadata and capacity assignment context. |
| Gold | `capacity_health_daily` | Daily health summary by capacity and workload. |
| Gold | `capacity_pressure_hotspots` | High-pressure windows, repeat offenders, and candidate tuning areas. |

## Key requirements

1. The pipeline must be restartable without duplicating records.
2. Every capture must include run metadata: run ID, capture timestamp, source, status, and row counts.
3. Tables must support incremental loading by time window.
4. Failures must be logged in a way that can be queried from the lakehouse.
5. Access to metrics data must follow tenant security and Fabric admin permissions.
6. The curated schema should remain stable even if raw source payloads change.

## Success measures

1. Capacity metrics land automatically in the lakehouse on schedule.
2. At least 30 days of historical data can be queried from the lakehouse.
3. Failed or missed ingestion runs are detectable within one schedule interval.
4. A Power BI semantic model can report daily capacity health, CU pressure, throttling, and workload contribution from the curated tables.
5. The implementation has clear operational ownership and documented recovery steps.

## Milestones

| Milestone | Outcome |
| --- | --- |
| Discovery | Confirm source access path, available fields, retention limits, and permissions. |
| Prototype ingestion | Capture a sample extract and land it in Bronze Delta format. |
| Lakehouse schema | Create Silver and Gold table definitions with incremental keys. |
| Orchestration | Schedule the capture pipeline and add run logging. |
| Reporting | Build initial semantic model and capacity health report. |
| Operational readiness | Document deployment, monitoring, recovery, and ownership. |

## Open questions

1. Which capacity or capacities should be included in the first release?
2. What history window is required: 30, 90, 180, or 365 days?
3. Should the solution capture all workspaces or only selected production workspaces?
4. What cadence is acceptable for ingestion: hourly, every few hours, or daily?
5. Who owns the lakehouse, pipeline, and downstream report after launch?
6. Are Activator alerts needed for throttling, sustained CU pressure, or failed ingestion?

## Recommended next step

Run a short discovery spike to confirm the exact source contract for the Capacity Metrics App data, including permissions, refresh cadence, available history, and whether the dataset or APIs provide the most reliable extraction path.