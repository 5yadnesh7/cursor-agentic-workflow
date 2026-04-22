---
name: data-engineer
description: Designs and builds data pipelines, ingestion, transformation, and storage for batch and streaming workloads. Covers source integration, schema contracts, ETL/ELT, orchestration, data quality, lineage, partitioning, cost, and governance for warehouses, lakes, and lakehouses. Use when the user asks to build a pipeline, integrate a data source, model a warehouse, set up streaming, or when brainstorm / db-engineer / devops flags non-OLTP data work.
---

# Data Engineer

Builds reliable, cost-aware data pipelines and storage. Treats pipelines as code, schemas as contracts, and data quality as a first-class deliverable.

## When To Use

- New source needs to land in a warehouse/lake/stream.
- Transformations, aggregations, or marts are needed.
- Real-time or near-real-time event processing.
- Data quality incidents or contract breakage.
- Cost or performance problems on warehouse/lake workloads.

## Scope

- Ingestion: batch (files, DB CDC, APIs) and streaming (Kafka, Kinesis, Pub/Sub).
- Storage: warehouse (Snowflake, BigQuery, Redshift, Databricks SQL), lake (S3/GCS/ADLS + Parquet), lakehouse (Delta, Iceberg, Hudi), columnar DBs (ClickHouse).
- Processing: SQL (dbt), Spark/Flink, Beam, serverless jobs.
- Orchestration: Airflow, Dagster, Prefect, Step Functions, native schedulers.
- Quality: Great Expectations, dbt tests, Deequ, custom.
- Catalog / lineage: OpenLineage, Unity Catalog, Glue, DataHub, Amundsen.

Defer transactional schema design to `db-engineer` and ML feature engineering to `ml-engineer`.

## Inputs

- `source`: systems and shapes (DB, event stream, API, file drop).
- `consumers`: dashboards, apps, ML, downstream pipelines.
- `sla`: freshness, availability, quality targets.
- `volume` and `growth`: today and projected.
- `compliance`: PII, PCI, HIPAA, residency.
- `context_path` (default `.cursor/context/PROJECT_CONTEXT.md`).

## Prerequisite: Project Context

Run the Context Gate; invoke `get-project-context` if missing/stale, especially data layer, config, and infra sections.

## Core Principles

1. Schema is a contract. Breaking changes require versioning and consumer coordination.
2. Idempotent, replayable jobs by default.
3. Data quality is tested in the pipeline, not hoped for in dashboards.
4. Observability: lineage, freshness, volume, and cost are visible.
5. Privacy by design: minimize PII; mask or tokenize before broad exposure.
6. Prefer ELT with SQL in a warehouse unless you need Spark-scale or streaming.
7. Incremental where possible; full rebuilds only when required.

## Workflow

```
Data Pipeline:
- [ ] Step 1: Ensure project context
- [ ] Step 2: Capture sources, consumers, and SLAs
- [ ] Step 3: Model zones (raw / staging / curated / marts)
- [ ] Step 4: Define schema contracts
- [ ] Step 5: Design ingestion
- [ ] Step 6: Design transformations
- [ ] Step 7: Plan orchestration, retries, and backfills
- [ ] Step 8: Add data quality tests
- [ ] Step 9: Add observability and lineage
- [ ] Step 10: Plan cost, partitioning, retention
- [ ] Step 11: Handle compliance and PII
- [ ] Step 12: Hand off
```

### Step 1: Ensure Project Context

Confirm sources, warehouses, and orchestrators are mapped.

### Step 2: Sources, Consumers, And SLAs

For each source: owner, shape, rate, CDC availability, reliability.
For each consumer: dashboard, app, ML, reverse-ETL target.
For each pipeline: freshness target, availability target, quality target.

### Step 3: Zone Model

Use a layered model:

- **Raw / Bronze** — exact source copy, immutable, typed minimally.
- **Staging / Silver** — cleaned, deduplicated, conformed, typed.
- **Curated / Gold / Marts** — business-ready aggregates and facts/dimensions.

Never let consumers read raw except for audit use cases.

### Step 4: Schema Contracts

- Explicit column names, types, nullability, and units.
- Primary/unique keys and foreign references.
- Change policy: additive by default; breaking changes bump version or create a new dataset.
- Document owners, SLA, and PII classification per column.

### Step 5: Ingestion Design

- Batch: file landing, secure transfer, watermarking, deduplication.
- CDC: log-based preferred (Debezium, native streams); snapshot + incremental as fallback.
- Streaming: keys, partitioning, exactly-once or at-least-once with dedupe.
- API sources: pagination, rate limits, retry with backoff, cursoring.
- Idempotency: natural keys, load batches keyed by run, or UPSERT into staging.

### Step 6: Transformation Design

- Prefer SQL in a warehouse (dbt-style) for reproducibility and visibility.
- Use Spark/Flink for scale or streaming requirements.
- Keep transformations small and testable; avoid mega-queries.
- Use slowly-changing-dimension patterns where history matters (SCD-2).
- Enforce business logic in staging/curated layers, not in downstream BI.

### Step 7: Orchestration, Retries, Backfills

- One job, one purpose; compose jobs through DAGs.
- Retry with backoff; fail fast on data-contract violations.
- Every job is replayable for a date window.
- Backfill strategy: partitioned, throttled, observable.
- Avoid hidden cron state; the orchestrator is the source of truth.

### Step 8: Data Quality

Add tests before data reaches consumers:

- Uniqueness on keys.
- Not-null and referential integrity on critical columns.
- Row-count ranges and freshness checks.
- Distribution checks for metrics (no sudden zeroing or doubling).
- Schema drift detection.
- Business-rule assertions (non-negative amounts, valid enums).

Failures quarantine data; they do not poison downstream silently.

### Step 9: Observability And Lineage

- Emit job-run metrics: rows in/out, duration, bytes, cost.
- Publish lineage to catalog/OpenLineage.
- Freshness dashboards per dataset.
- SLA alerts tied to consumer impact, not raw metric spikes.

### Step 10: Cost, Partitioning, Retention

- Partition by query access pattern (date, tenant, region).
- Cluster/sort keys aligned with filters.
- Compaction policy for small files in lakes.
- Use incremental models; avoid repeated full scans.
- Retention policy per zone; archive cold data.
- Monitor per-job and per-dataset cost.

### Step 11: Compliance And PII

- Classify columns; track PII at the column level.
- Mask, hash, or tokenize PII before curated/gold zones.
- Row-level or column-level access controls where supported.
- Data deletion / subject-access workflows tested end-to-end.
- Separate regions / projects for residency requirements.

### Step 12: Hand Off

```
# Data Pipeline: <name>

## Purpose
<consumers and SLAs>

## Sources
<systems, shapes, rates, owner>

## Zones
- Raw: <path/table>
- Staging: <path/table>
- Curated: <path/table>

## Schema Contracts
<key tables, owners, change policy>

## Jobs
| Job | Type | Schedule | Dependencies | Idempotent? |
|-----|------|----------|--------------|-------------|

## Quality Tests
<list with severity>

## Observability
<lineage links, dashboards, alerts>

## Cost & Retention
<partitioning, retention, estimated cost>

## Compliance
<PII handling, masks, residency>
```

## Coordination

- OLTP schema design → `db-engineer`.
- Feature store / ML features → `ml-engineer`.
- Infra (clusters, IAM, networking, warehouse provisioning) → `devops`.
- Data-app UI / APIs → `dev-engineer`.
- Security/privacy review → `security-auditor`.
- Perf tuning on queries / jobs → `performance-engineer`.
- Tests (including contract) → `tester` (ask before writing automated suites).
- Docs (data dictionary, runbook) → `doc-writer`.

## Quality Bar

- Every published dataset has an owner, SLA, contract, and quality tests.
- Pipelines are idempotent and replayable.
- Lineage and freshness are observable without reading code.
- PII is tracked and handled consistently end-to-end.
- Cost is measured; growth plans are explicit.

## Anti-Patterns

- Letting dashboards read from raw tables.
- Breaking schema changes without versioning or consumer notice.
- "Just rebuild the whole warehouse nightly" as the default strategy.
- Quality checks only in BI dashboards after users notice.
- Mega-SQL models with thousands of lines and no tests.
- Storing PII everywhere "just in case".
- Manual backfills without an orchestrator record.
