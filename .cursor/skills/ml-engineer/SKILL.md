---
name: ml-engineer
description: Designs and ships machine-learning systems end-to-end: problem framing, data and feature engineering, model selection and training, offline and online evaluation, deployment, monitoring for drift and quality, and responsible-AI checks. Covers classical ML, deep learning, and LLM / RAG / agent systems. Use when the user asks to build, tune, deploy, or evaluate an ML or LLM feature, or when brainstorm / dev-engineer / data-engineer flags ML-specific work.
---

# ML Engineer

End-to-end ML systems skill. Treats ML as a software system first, a modeling problem second. Optimizes for correctness, reproducibility, safety, and maintainability.

## When To Use

- A user story requires prediction, ranking, generation, classification, search, or recommendation.
- An existing ML system needs retraining, evaluation, or monitoring.
- An LLM feature (chat, RAG, tool-use, agent) must be built or improved.
- Model quality, drift, or safety concerns are raised in production.

## Scope

- Classical ML: regression, classification, clustering, time series, tree ensembles.
- Deep learning: vision, NLP, audio, multimodal.
- LLM systems: prompt design, RAG, tool/function calling, agents, fine-tuning, evals.
- MLOps: feature stores, training pipelines, registries, deployment, monitoring.
- Responsible AI: bias, fairness, privacy, safety, transparency.

Defer data-plumbing to `data-engineer`, infra to `devops`, and API/UI to `dev-engineer`.

## Inputs

- `problem`: predict/generate/rank what, for whom, measured how.
- `data`: sources, labels, volumes, quality, PII status.
- `constraints`: latency, cost, on-device vs server, compliance, safety.
- `baseline`: current heuristic or existing model.
- `context_path` (default `.cursor/context/PROJECT_CONTEXT.md`).

## Prerequisite: Project Context

Run the Context Gate; invoke `get-project-context` when missing or stale, especially for data layer, services, and deploy surface.

## Core Principles

1. Start with a measurable business metric and a simple baseline.
2. Data quality beats model complexity.
3. Reproducible training runs: code, data, config, seeds, and environment pinned.
4. Offline metrics are a proxy; validate online with a controlled rollout.
5. Monitor data and prediction drift, not just infra.
6. Safety, privacy, and fairness are not optional steps.
7. Own the lifecycle: retraining, rollback, and retirement.

## Workflow

```
ML System:
- [ ] Step 1: Ensure project context
- [ ] Step 2: Frame the problem and metrics
- [ ] Step 3: Assess data readiness
- [ ] Step 4: Build a strong baseline
- [ ] Step 5: Feature / input design
- [ ] Step 6: Model selection and training
- [ ] Step 7: Evaluation (offline + slice + safety)
- [ ] Step 8: Packaging and deployment
- [ ] Step 9: Online validation and rollout
- [ ] Step 10: Monitoring and lifecycle
- [ ] Step 11: Documentation and handoff
```

### Step 1: Ensure Project Context

Confirm data sources, existing services, and deploy surface are mapped.

### Step 2: Frame The Problem And Metrics

- Restate the business objective in one sentence.
- Define primary metric (what you optimize) and guardrail metrics (what you must not break).
- Define the prediction unit, cadence, and latency budget.
- Identify acceptable error modes and unacceptable ones.

For LLM features, add:

- Target behaviors and refusal criteria.
- Tool-use boundaries.
- Safety and privacy constraints.

### Step 3: Assess Data Readiness

- Coverage, labels, class balance, leakage risk.
- Temporal structure: splits by time where relevant to avoid leakage.
- PII and compliance classification.
- Bias and representativeness audit.
- Missing data strategy; outlier handling.

Hand off pipeline and storage needs to `data-engineer`.

### Step 4: Strong Baseline

Always build a non-ML or simple-ML baseline first:

- Heuristics, rules, or linear models.
- For LLM: a well-crafted prompt with minimal scaffolding.

The baseline defines the bar the complex model must clear.

### Step 5: Feature / Input Design

- Clear, documented, reproducible feature definitions; versioned.
- Training/serving parity; use a feature store or shared library to avoid skew.
- For text/vision/audio: preprocessing consistency between offline and online.
- For LLM/RAG: chunking strategy, embedding model, index, retrieval hyperparameters.

### Step 6: Model Selection And Training

- Start simple; scale up only when the baseline and data justify it.
- Track every run (code, data snapshot, config, seed, metrics) with an experiment tracker.
- Keep training deterministic where possible; record non-determinism sources.
- Use cross-validation or time-based splits honestly; no peeking.
- Tune hyperparameters with a budgeted search, not indefinite tweaking.

For fine-tuning LLMs:

- Prefer prompting / RAG / tool-use first.
- Fine-tune only with clear, proven benefit and curated data.
- Preserve eval suites across methods for apples-to-apples comparison.

### Step 7: Evaluation

Offline:

- Primary metric vs baseline.
- Slice metrics (by segment, time, geography, class, cohort) to expose hidden failures.
- Calibration where probabilities matter.
- Robustness: noisy inputs, shifted distributions, adversarial prompts for LLMs.

Safety and fairness:

- Bias and disparity checks on protected or sensitive slices.
- Harmful content / jailbreak evals for LLM systems.
- Privacy evals (memorization, leakage) for generative models.

Human evaluation:

- Rubric-based blind comparisons for generative tasks.
- Inter-rater agreement tracked.

### Step 8: Packaging And Deployment

- Model registry with versions, lineage, and sign-off.
- Serving path: batch, real-time, on-device, or agent runtime.
- Input/output contracts; validation at the boundary.
- Latency and memory budgets respected.
- Feature flags to toggle models quickly.

### Step 9: Online Validation And Rollout

- Shadow traffic first; compare predictions without impact.
- A/B or interleaved experiments with predefined success criteria and duration.
- Guardrail metrics watched; auto-stop on breach.
- Progressive rollout with rollback path.

### Step 10: Monitoring And Lifecycle

Monitor:

- Input drift (feature distributions).
- Prediction drift (output distributions).
- Quality metrics from feedback loops or labelers.
- Fairness/safety metrics over time.
- Latency, error rate, cost per prediction.

Lifecycle:

- Retraining policy (cadence, triggers).
- Retirement and replacement plan.
- Incident response for model failures (tied to `incident-commander`).

### Step 11: Documentation And Handoff

Produce a model card and system doc via `doc-writer`:

```
# Model Card: <name>

## Purpose
<what it predicts, for whom, at what cadence>

## Data
- Sources, time window, size, splits, PII, biases known

## Method
- Features / inputs, model family, training setup

## Evaluation
- Offline metrics vs baseline
- Slice metrics
- Safety / fairness metrics

## Deployment
- Serving path, latency, cost, owners

## Monitoring
- Drift, quality, guardrails, alerts

## Limits
- Known failure modes, out-of-scope uses

## Lifecycle
- Retraining, rollback, retirement
```

## LLM / RAG / Agent Specifics

- Start with the smallest capable model; scale only when evals demand it.
- RAG: retrieval quality is usually the ceiling; tune it before prompting tricks.
- Ground outputs; cite sources when the product allows.
- Define refusal and safety policies; build evals that cover them.
- Log prompts, retrieved context, tool calls, and outputs (redact PII) for debugging.
- Budget: track token and tool-call costs per request.

## Coordination

- Data pipelines, feature store plumbing → `data-engineer`.
- Infra, GPU, serving stacks → `devops`.
- Application integration (APIs, UI) → `dev-engineer`.
- Security/privacy deep dive → `security-auditor`.
- Performance tuning of serving → `performance-engineer`.
- Incident response on model or safety failures → `incident-commander`.
- Docs and model cards → `doc-writer`.

## Quality Bar

- The primary metric and guardrails are explicit and measured.
- The new model beats a documented baseline, or it is not shipped.
- Every production prediction path has drift and quality monitoring.
- Safety, fairness, and privacy evals exist and pass.
- Reproducibility: a new engineer can retrain the model from the repo alone.

## Anti-Patterns

- Reporting only aggregate accuracy while per-slice metrics regress.
- Training/serving skew from inconsistent feature code.
- Shipping without an offline baseline comparison.
- Fine-tuning an LLM before exhausting prompting + RAG.
- Monitoring only infra metrics, not data and predictions.
- Unbounded prompt/context growth with no cost controls.
- "The model is a black box" used as an excuse to skip evaluation slices.
