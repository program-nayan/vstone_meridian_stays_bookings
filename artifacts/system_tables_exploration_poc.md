# Phase 0: System Tables Exploration Proof of Concept

**Objective**
As part of Phase 0 for Project Meridian, I researched and explored the Databricks `system` catalog to identify which system tables are accessible in my Free Edition workspace. My goal is to understand how I will use these tables to build my Phase 7 operations dashboard and prove the platform's health to the CDO.

## Accessible System Tables and My Implementation Plan

Based on the Databricks documentation and my workspace exploration, I have identified the following system tables that I will use to answer the required operational questions:

*   **`system.billing.usage`**
    *   *Question:* What consumes compute?
    *   *My Plan:* I will query this table to track serverless DBU consumption. Since the Free Edition is exclusively serverless, this will be the primary source for the cost and usage page of my dashboard.
*   **`system.lakeflow.jobs` and `system.lakeflow.job_run_timeline`**
    *   *Question:* Did the last run succeed?
    *   *My Plan:* I will use these tables to monitor my end-to-end job runs. This will also allow me to capture and visualize the deliberate job failure I need to trigger in Phase 5.
*   **`system.lakeflow.job_tasks` and `system.lakeflow.job_task_run_timeline`**
    *   *Question:* Which task is slowest? How long do pipeline updates take?
    *   *My Plan:* By querying task-level execution logs, I can measure the duration of my DLT pipeline updates and identify any performance bottlenecks in my workflow.
*   **`system.query.history`**
    *   *Question:* Which queries are slow?
    *   *My Plan:* I will use this table to track SQL query durations across my workspace, specifically monitoring the performance of the queries executed from the SQL Editor and dashboards.
*   **`system.access.table_lineage` and `system.access.column_lineage`**
    *   *Question:* Where does Gold data come from?
    *   *My Plan:* I will use these lineage tables to trace the dependencies of my Gold models back through the Silver and Bronze layers, providing clear data provenance.

## Limitations and Alternatives (Free Edition)

*   **Cluster Telemetry:** Because the Databricks Free Edition relies entirely on serverless compute, tables related to classic cluster metrics (like `system.compute.clusters`) are mostly irrelevant to my project. I will focus my dashboard entirely on serverless and job-specific telemetry.
*   **Data Quality Monitoring:** To answer the question "Is data quality getting worse?", I found that relying solely on system tables might be insufficient or restricted. Following the project brief's guidance, I will query my pipeline's event log directly to extract and report the expectation pass and fail counts for my operations dashboard. 