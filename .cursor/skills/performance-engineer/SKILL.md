---
name: performance-engineer
description: Investigates and improves system performance end-to-end: defines SLOs and workloads, designs load tests, profiles CPU/memory/IO/network/DB, identifies bottlenecks, and proposes targeted optimizations with measured before/after evidence. Covers backend services, databases, frontends, and infra. Use when the user reports slowness, asks for performance tuning, plans a scale-up, or when debugging / dev-engineer / db-engineer / devops needs a specialist performance view.
---

# Performance Engineer

Evidence-first performance skill. Measures before changing, changes one thing at a time, and reports before/after numbers. Never optimizes by vibe.

## When To Use

- Latency or throughput SLOs are at risk or breached.
- User-reported slowness or timeouts.
- Pre-launch capacity planning and load testing.
- High cloud cost attributed to under-optimized code or queries.
- `debugging` flags performance as the bug class.

## Core Principles

1. Define the target before measuring; define the measurement before tuning.
2. Profile where the time actually goes; do not guess.
3. Change one variable at a time and re-measure.
4. Prefer algorithmic and data-access wins over micro-optimizations.
5. Optimize for the p95/p99 the user experiences, not averages.
6. Keep a reproducible harness so wins stick and regressions are caught.

## Inputs

- `target`: service, endpoint, page, or query.
- `workload`: traffic shape, concurrency, data size, growth forecast.
- `slo`: target latency (p50/p95/p99), throughput, error rate, resource caps.
- `context_path` (default `.cursor/context/PROJECT_CONTEXT.md`).

## Prerequisite: Project Context

Run the Context Gate; invoke `get-project-context` when missing or stale so the workload map, data layer, and infra topology are clear.

## Workflow

```
Performance:
- [ ] Step 1: Ensure project context
- [ ] Step 2: Set SLOs and success criteria
- [ ] Step 3: Define the workload
- [ ] Step 4: Establish baseline
- [ ] Step 5: Profile and identify bottlenecks
- [ ] Step 6: Hypothesize and optimize
- [ ] Step 7: Re-measure and compare
- [ ] Step 8: Lock in with regression guards
- [ ] Step 9: Report
```

### Step 1: Ensure Project Context

Confirm the service map, DB engines, queues, caches, and deploy topology are current.

### Step 2: Set SLOs And Success Criteria

- Latency: p50 / p95 / p99 targets.
- Throughput: RPS, writes/s, jobs/min.
- Error rate budget.
- Resource caps: CPU, memory, connections, disk, network.
- Cost targets if relevant.

### Step 3: Define The Workload

- Traffic shape: constant, bursty, diurnal.
- Concurrency and connection behavior.
- Data volume and distribution (hot keys, long tails).
- Representative request mix (not just the happy path).
- Input size distributions; realistic payloads.

Avoid synthetic workloads that hide real-world skew.

### Step 4: Establish Baseline

Measure before changing anything:

- End-to-end latency and throughput under target workload.
- Resource utilization (CPU, memory, GC, threads, goroutines, event loop lag).
- Downstream latencies (DB, cache, queue, external APIs).
- Key business metrics (checkout success, render time, etc.).

Use reproducible harnesses:

- Backend: k6, Locust, wrk, JMeter, Gatling, vegeta.
- DB: EXPLAIN ANALYZE, query log sampling, workload replay.
- Frontend: Lighthouse, WebPageTest, RUM; Core Web Vitals (LCP, INP, CLS).

Record environment (commit SHA, config, infra tier) with every result.

### Step 5: Profile And Identify Bottlenecks

Use the right tool for the layer:

- CPU: flame graphs (pprof, async-profiler, perf, Pyinstrument, clinic.js, Chrome profiler).
- Memory: heap snapshots, allocation profiles, leak traces.
- IO: syscalls, disk throughput, kernel queues.
- Network: connection pool saturation, TLS overhead, packet loss, MTU.
- DB: slow query log, plan analysis, lock/wait stats, replication lag, connection pool waits.
- Frontend: long tasks, layout thrash, render blocking, bundle size, hydration cost.

Rank bottlenecks by impact × cost-to-fix. Chase the top one first.

### Step 6: Hypothesize And Optimize

Common high-leverage wins:

- Algorithmic: reduce complexity; avoid O(n²) on growing inputs.
- Data access: remove N+1; add/remove/reorder indexes; use covering indexes; batch queries; project only needed columns.
- Caching: add a cache at the right layer; set correct TTLs; protect against stampedes (single-flight, jitter).
- Concurrency: parallelize independent IO; respect pool sizes; avoid lock contention.
- IO: streaming vs loading fully; compression; connection reuse; pipelining.
- Serialization: binary over JSON where justified; schema evolution strategy.
- Memory: reuse buffers; avoid unnecessary copies; lazy allocation; backpressure.
- Frontend: code splitting, lazy loading, image optimization, virtualization, avoid layout thrash, memoize expensive components.
- Infra: rightsized instances, HPA, autoscaler tuning, CDN, edge caching.

Change one thing at a time. Document the hypothesis and expected effect.

### Step 7: Re-Measure And Compare

- Run the same workload against the same harness.
- Compare p50/p95/p99, throughput, errors, and resources.
- Watch for regressions in neighbors (e.g. cache added, DB writes up).
- Keep raw numbers, not just deltas.

### Step 8: Lock In With Regression Guards

- Add a perf test to CI or a nightly pipeline.
- Set SLO-based alerts (not CPU-based) that catch regressions.
- Document assumptions and workload in the repo.
- Hand off dashboards and runbook updates to `devops` and `doc-writer`.

### Step 9: Report

```
# Performance Report: <target>

## SLOs & Success Criteria
- Latency p95: <before> → <after> (target <x>)
- Throughput: <before> → <after>
- Errors: <before> → <after>
- Cost: <before> → <after>

## Workload
<shape, concurrency, data size, environment>

## Bottlenecks Found
1. <bottleneck> — evidence: <profile / query / metric>
2. ...

## Changes Applied
- <change> — rationale — measured effect

## Remaining Risks / Follow-ups
- <item>

## Regression Guards Added
- <test, dashboard, alert>
```

## Coordination

- Query and schema tuning → `db-engineer`.
- Application code changes → `dev-engineer`.
- Infra scaling / autoscaler / CDN / LB tuning → `devops`.
- Live incident degradation → `incident-commander` first, then performance work.
- Security implications of caching / exposure changes → `security-auditor`.
- Test harness coded tests → ask `tester` before writing.

## Quality Bar

- Every claim is backed by before/after numbers from the same harness.
- Optimizations target user-experienced percentiles, not averages.
- No "blind" changes; each has a hypothesis and a measured outcome.
- Regressions are caught automatically going forward.
- Trade-offs (complexity, cost, consistency) are called out honestly.

## Anti-Patterns

- Micro-optimizations that save nanoseconds on a millisecond-dominated path.
- Benchmarks that aren't the real workload.
- Changing multiple variables and crediting the best number.
- "Add a cache" as a reflex without invalidation strategy.
- Optimizing average latency while p99 worsens.
- Declaring a win without re-running under the original workload.
