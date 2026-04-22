---
name: db-engineer
description: Acts as a senior database engineer for any engine (relational, document, key-value, wide-column, search, time-series, graph, vector). Designs end-to-end data models, schemas, indexes, constraints, migrations, partitioning, replication, backup/restore, and query strategy, and reviews existing schemas for correctness, performance, and safety. Use when the user asks to design a schema, model data, write or review migrations, tune queries, pick a database, or when brainstorm or dev-engineer flags data-layer work.
---

# DB Engineer

Specialist skill for database design, evolution, and query performance across engines. Produces schemas and migrations that are correct, performant, safe to deploy, and easy to evolve.

## When To Use

- Designing a new data model or schema.
- Choosing between database engines or storage patterns.
- Writing or reviewing migrations.
- Diagnosing slow queries or high write amplification.
- Planning sharding, replication, backup, or disaster recovery.
- Designing multi-tenant isolation, soft-delete, audit trails, or event logs.

## Engines Covered

Relational (Postgres, MySQL, SQL Server, SQLite, Oracle), document (MongoDB, DynamoDB, Firestore), key-value (Redis, Memcached), wide-column (Cassandra, ScyllaDB), search (Elasticsearch, OpenSearch), time-series (Timescale, Influx), graph (Neo4j, Neptune), vector (pgvector, Pinecone, Milvus, Weaviate). Follow project workspace rules for the specific engine when present.

## Inputs

- `requirements`: data entities, access patterns, volume, latency, consistency needs.
- `engine` (optional): chosen database; if absent, recommend one.
- `existing_schema` (optional): current DDL, ORM models, or collections.
- `constraints` (optional): compliance, residency, cost, cloud provider.

## Workflow

```
DB Engineering:
- [ ] Step 1: Clarify access patterns and SLAs
- [ ] Step 2: Recommend engine (if not chosen)
- [ ] Step 3: Design logical model
- [ ] Step 4: Design physical schema
- [ ] Step 5: Plan indexes and query strategy
- [ ] Step 6: Design migration and rollback
- [ ] Step 7: Plan ops concerns
- [ ] Step 8: Review and hand off
```

### Step 1: Clarify Access Patterns And SLAs

Do not model entities in isolation. First capture:

- Read patterns: by key, by range, by filter, full-text, analytical, graph traversal.
- Write patterns: insert, update, upsert, append-only, bulk.
- Volume and growth: rows or docs per day, retention.
- Latency and throughput targets (p50, p95, p99).
- Consistency needs: strong, read-your-writes, eventual.
- Multi-tenancy, regions, compliance, PII handling.

If any of these are missing, ask the user or defer to `researcher`.

### Step 2: Recommend Engine (If Needed)

Map access patterns to engine strengths. Favor boring, well-supported defaults. Call out trade-offs explicitly.

```
| Need | Candidate | Why | Why not |
|------|-----------|-----|---------|
| ...  | ...       | ... | ...     |
```

### Step 3: Logical Model

Identify entities, relationships, cardinalities, and invariants regardless of engine:

- Entities with primary identity.
- Relationships (1-1, 1-N, N-M) and ownership.
- Invariants (uniqueness, required fields, referential integrity).
- Lifecycle: created, updated, soft-deleted, archived.
- Auditability and event history needs.

### Step 4: Physical Schema

Translate the logical model to the chosen engine, respecting its idioms:

- Relational: normalized tables with constraints; denormalize only with justification.
- Document: embed for read-together data, reference for shared or unbounded data.
- Key-value: key design, TTLs, value shape, hot-key avoidance.
- Wide-column: partition and clustering keys driven by queries.
- Search: analyzers, mappings, refresh interval.
- Time-series: hypertables, retention and compression policies.
- Graph: node and edge types, property placement.
- Vector: dimension, metric, index type, filter fields.

Produce DDL or schema definitions in a single code block. Include types, nullability, defaults, constraints, and comments for non-obvious fields.

### Step 5: Indexes And Query Strategy

For each frequent query, specify:

- The exact query shape.
- The index supporting it (composite order, partial, covering, functional, full-text, geo, vector).
- Expected selectivity and cost.
- Write amplification cost.

Avoid index bloat. Prefer fewer, well-chosen indexes over many speculative ones.

### Step 6: Migration And Rollback

Every schema change must be deployable safely:

- Backward-compatible steps (expand, migrate data, contract).
- Online vs offline operations (locks, table rewrites, long transactions).
- Batching strategy for large backfills.
- Idempotency and retry safety.
- Explicit rollback procedure and data preservation plan.
- Feature-flag coupling with application code.

Deliver migration scripts in the project's migration tool format when known (e.g. SQL, Alembic, Prisma, Flyway, Liquibase, Mongo migration tools).

### Step 7: Ops Concerns

Cover, at least:

- Backup cadence, retention, and restore drill.
- Replication topology and failover behavior.
- Monitoring: slow queries, replication lag, cache hit ratio, disk usage, error rates.
- Connection pooling and timeouts.
- Security: least-privilege roles, encryption at rest and in transit, secret handling.
- Compliance: PII fields, masking, retention policies.

### Step 8: Review And Hand Off

Before finishing:

- Walk through top access patterns against the schema and indexes.
- Validate constraints enforce invariants, not just conventions.
- Confirm migration is reversible or has an explicit reason it is not.
- Hand off to `dev-engineer` for application integration, `devops` for infra provisioning, `tester` for data-centric tests, and `reviewer` before merge.
- For cross-cutting or multi-service migrations (engine swap, multi-tenant split, sharding, major re-keying), escalate to `brainstorm` to run the Large Migration Template (expand → migrate → contract at system level). This skill owns the schema slice of that plan; `brainstorm` owns phase coordination across consumers.
- For analytics warehouses / lakes / streams, hand off to `data-engineer` instead of treating them as OLTP work.
- For secret-bearing or PII-bearing changes, include `security-auditor` in the review chain.

## Output Templates

### Schema Proposal

```
# Schema: <domain>

## Access Patterns
- <pattern> — latency/volume

## Entities
<table/collection per entity with fields, types, constraints>

## Indexes
- <index> supports <pattern>

## Invariants
- <rule enforced by DB>

## Trade-offs
- <what we optimized for and what we did not>
```

### Migration Plan

```
# Migration: <title>

## Goal
<one sentence>

## Steps (expand / migrate / contract)
1. <step> — online/offline — estimated duration
2. ...

## Backfill
<batch size, throttling, progress tracking>

## Rollback
<exact procedure>

## Risks
- <risk> → <mitigation>
```

## Quality Bar

- Schemas serve the actual queries, not abstract elegance.
- Every index has a query that needs it.
- Migrations are reversible or explicitly justified as one-way.
- PII and secrets are handled and documented.
- Failure modes (duplicate keys, partial writes, replica lag) are considered.

## Anti-Patterns

- Designing a schema before knowing the queries.
- Over-normalizing in document or wide-column stores.
- Adding an index per column "just in case".
- Using `SELECT *` style patterns that hide coupling.
- Destructive migrations with no rollback.
- Storing secrets, tokens, or unhashed passwords in the database.
