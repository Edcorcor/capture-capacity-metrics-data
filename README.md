# Capture Capacity Metrics Data

This project captures data from the Microsoft Fabric Capacity Metrics App semantic model and stores it in a Fabric lakehouse for historical analysis, operations reporting, and capacity-health monitoring.

## What this includes

- A Fabric notebook that connects to the Capacity Metrics semantic model through Semantic Link.
- Configurable exports for discovered semantic model tables or named DAX queries.
- Delta table writes into an attached lakehouse.
- Capture metadata and a run-log table for operational monitoring.
- A project brief that describes the scope, architecture, milestones, and open decisions.

## Repository contents

| File | Purpose |
| --- | --- |
| `capture-capacity-metrics-data.ipynb` | Fabric notebook for extracting Capacity Metrics App data and writing it to a lakehouse. |
| `project-brief.md` | Project brief for customer or stakeholder review. |

## How to use the notebook

1. Create or select a Fabric workspace with active capacity.
2. Create a lakehouse for captured capacity metrics.
3. Upload `capture-capacity-metrics-data.ipynb` to the workspace.
4. Attach the lakehouse as the notebook's default lakehouse.
5. Set the notebook parameters:
   - `semantic_model_workspace`: workspace that contains the Capacity Metrics App semantic model. Leave blank to use the current workspace.
   - `semantic_model_name`: usually `Fabric Capacity Metrics`, unless the model was renamed.
   - `tables_to_export`: optional list of model tables to export. Leave empty to discover visible tables.
   - `dax_queries`: optional named DAX queries for curated extracts.
   - `write_mode`: use `append` for scheduled captures.
6. Run the notebook once interactively to validate permissions and table output.
7. Schedule it with a Fabric pipeline or notebook schedule.

## Output tables

By default, exported tables use the `capacity_metrics_` prefix. The notebook also writes `capacity_metrics_capture_run_log`, which records each export attempt, target table, row count, status, and error text.

## Notes

The notebook depends on Microsoft Fabric Semantic Link and must run in Fabric. Access to the Capacity Metrics App semantic model depends on the user's Fabric and Power BI permissions.