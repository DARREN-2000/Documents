# AI Platform Design

**Author:** Darren Morris · **Scope:** Task 1 design and context for implemented Option D

**Reading guide:** This document is intentionally limited to **four print pages**; the checked-in [`DESIGN.pdf`](DESIGN.pdf) is the verified A4 rendering. Quantitative evidence is in [`docs/DATA_ANALYSIS.md`](docs/DATA_ANALYSIS.md); derivations, metric semantics, and operating details are in [`docs/DESIGN_APPENDIX.md`](docs/DESIGN_APPENDIX.md).

## Page 1 of 4 — Context, priorities, and cost governance

### Executive summary

The platform routes six workloads through LiteLLM on ECS Fargate to Anthropic models on Bedrock Frankfurt and one external Zero-Data-Retention provider. Langfuse stores request traces; Postgres holds spend/key metadata; Datadog receives aggregate operational metrics only. I assume AdvisorChat and KYC are quality-sensitive, the daily spend export is up to 24 hours stale, and raw prompts/outputs must remain in Langfuse/ClickHouse.

The supplied data sets the priorities: known spend is **$36,468.89 over 30 days**; daily spend rises from **$589.25** before 17 November to **$1,931.49** afterwards (**3.28×**); DevAgent represents **55.3%** of spend; and a personal production key has no team owner. One high-volume KYC row has missing cost. The calculations and eleven findings are reproducible in the data appendix.

The target architecture separates:

1. **Control plane:** LiteLLM admission, policy, routing, budgets, and tool authorization.
2. **Telemetry plane:** Langfuse remains the request-level source of truth; Option D derives privacy-safe SLIs without copying traces.
3. **Governance plane:** declarative workload registration defines owner, data class, providers/regions, budget, model policy, and tool permissions.

### Cost governance

A daily CSV is suitable for reconciliation but too stale for enforcement. The gateway should maintain near-real-time per-key and per-team token/cost counters in Redis or Postgres. Admission policy evaluates those counters before the provider call:

| Budget state | Default action | Customer-facing / regulated routes |
|---|---|---|
| <75% | Allow | Allow |
| 75–90% | Notify owner/platform | Notify only |
| 90–100% | Reduce concurrency; approved downgrade | Escalate; no silent downgrade |
| ≥100% | Queue/block non-critical batch/dev | Explicit owner decision |

Every route also gets request input/output limits, maximum agent steps, per-key RPS, and in-flight concurrency caps. These controls stop a runaway request or loop before delayed aggregate spend can react. DigestBot and Research are reasonable downgrade/defer candidates; AdvisorChat and KYC are not silently changed because substitution can alter refusal, accuracy, and compliance evidence.

First attribute the 17 November step change, replace the personal key with a team identity, assign every key a budget owner, and reconcile live counters against finance. **Option B** would enforce admission in the gateway; implemented **Option D** verifies its effect but never blocks traffic.

<div style="page-break-after: always;"></div>

## Page 2 of 4 — Data governance, agent safety, and self-service

### Data governance

Routing is driven by workload data classification—not a developer-entered alias. A registration declares owner, environment, data class, retention, approved providers/regions, budget, and model set. The gateway evaluates policy and records model/version, routing decision, prompt-template provenance, and configuration for reproducibility.

PII-sensitive AdvisorChat and KYC use approved in-region paths by default. The external provider is allowed only where its ZDR contract and workload classification permit it. Raw prompts/outputs remain in the single Langfuse/ClickHouse store with scoped access, encryption, and retention. Datadog receives aggregates only. Secrets come from Secrets Manager; workload identities are least-privilege and environment-scoped.

### Agentic safety

DevAgent is both the largest spender and write-capable through MCP, so it has the highest combined cost and blast-radius risk. Every tool is classified **read**, **write**, or **destructive**. Enforcement sits between intent and execution:

- read tools use scoped identities and ordinary audit logging;
- write tools require resource scopes and per-session operation budgets;
- destructive actions require approval, idempotency where supported, and an actor/tool/resource audit record;
- sessions support kill/reset, maximum steps, timeouts, and token/tool budgets.

Agent identities are unique per workload/environment; shared privileged keys are rejected. Prompt content is untrusted input, never authority to expand tool scopes. **Option A** would be this in-path policy point; Option D observes errors, latency, and runaway-token signatures out of path.

### Self-service paved road

Onboarding is a pull request against a declarative workload registry. CI rejects registrations without a team and budget owner, data class, provider/region policy, retention, and non-personal service identity. It validates least-privilege IAM, bounded telemetry dimensions, route/model vocabulary, and destructive-tool approvals. After merge, automation provisions policy and registers dashboards, alerts, and budget defaults.

This prevents the supplied-data defects structurally: an untagged personal key cannot reach production, and a route cannot create unbounded Datadog series. Teams own workload quality, budget, and route SLOs; Platform owns defaults and runtime; Security/DPO reviews privileged tools and egress; Finance owns the budget envelope and reconciliation source. **Option C** would implement this control surface.

<div style="page-break-after: always;"></div>

## Page 3 of 4 — Observability and implemented Option D

### Architecture and privacy boundary

Option D reads a delayed, closed Langfuse window, parses and normalizes records, folds them into streaming accumulators, and exports OTLP metrics to a shared collector. It stores only a watermark and bounded overlap IDs; there is no second trace store. It is outside the inference path, so failure reduces observability rather than customer availability.

Only governed values can become labels: `team`, `route`, `model_family`, `env`, `outcome`, `error_category`, `provider_region`, and a bounded pipeline `dimension`. Unknown values fold to `other`; blank ownership folds to `unattributed`. Before export, one registry validates metric name, type, unit, allowed dimension set, keys, and values. Prompts, outputs, request/user IDs, key aliases, trace IDs, and free-text provider messages have no export path.

### SLIs and alerting

| Signal | Definition / use |
|---|---|
| Latency / TTFT | Backend-composable histograms; TTFT only for successful streaming |
| Reliability | Request/error counters by bounded outcome/category |
| Cost | USD, requests, input/output/cache tokens; absent vs invalid cost integrity |
| Agent efficiency | Completion/input ratio and empty completions |
| Quality proxy | Route completion-length anomaly; investigation only |
| Telemetry trust | Source/export success, freshness, malformed/quarantined, truncation, unknown dimensions, checkpoint conflict, series count |

Customer-facing availability pages through paired fast/slow SLO burn-rate monitors. Cost drift, token ratio, and quality proxies create investigation signals. Stale/no-data or failed source/export pages only when it creates an operational blind spot.

### Correctness and scale

Aggregation is single-pass. Histograms and quality reservoirs are bounded; exact in-window de-duplication is capped by required `max_records`, and records without an id fail closed rather than becoming non-idempotent. Pagination detects repeated cursors/pages and contradictory empty continuation pages, validates response shape, retries bounded transient failures, and fails rather than checkpointing incomplete reads. A grace delay captures most late arrivals; overlap re-reads are de-duplicated.

Production uses DynamoDB conditional checkpoint writes; the file store is development-only. Delivery is explicitly **at-least-once**: checkpoint concurrency prevents stale watermark overwrite, but export and checkpoint are not atomic. One task owns each window, conflicts fail the run, and backend duplicate tolerance is required. A pre-export lease/idempotency key is the next hardening step.

Cardinality is bounded by an enumerated registry. The primary product is `team × route × model_family × env`; cost adds bounded provider region. CI verifies vocabulary budgets and monitor thresholds. OTLP uses strict JSON, deadlines, and retries. The container is non-root/read-only and referenced by immutable digest. Exact formulas and residual risks are in the appendix.

<div style="page-break-after: always;"></div>

## Page 4 of 4 — Model sourcing, roadmap, alternatives, and decisions

### Model sourcing

Managed models remain the default. Known spend annualizes to about **$438k**, while the recent run-rate is about **$705k**; 94% is concentrated in two frontier families. Self-hosting is not automatically cheaper: economics depend on sustained utilization, validated quality, capacity engineering, patching, and availability ownership.

Pilot a Bedrock-managed open-weight model for low-risk batch workloads (DigestBot and Research), with offline gates and an online canary. Keep frontier models for AdvisorChat, KYC, and DevAgent until task evidence supports change. Reconsider self-hosting only when one workload is large and steady enough for roughly >60% GPU utilization and an open model passes quality/compliance evaluation.

### Six-month direction

1. **Month 0–1:** deploy Option D; attribute the spend jump; replace the personal key; add live counters/owners.
2. **Month 1–2:** implement Option B admission using those counters; reconcile with finance.
3. **Month 2–4:** deliver Option C registry/CI onboarding and migrate workloads.
4. **Month 3–5:** implement Option A tool policy for DevAgent, including destructive approvals and session budgets.
5. **Month 4–6:** run the managed open-weight pilot; add asynchronous semantic evaluation and a durable quality baseline.

Success means every workload has an owner/policy/budget, no personal keys exist, telemetry is fresh and privacy-safe, pages are SLO-based, and model changes have cost/quality evidence.

### Rejected alternatives

- **Direct ClickHouse reads:** couples to Langfuse internals and bypasses the supported boundary.
- **Second trace store:** duplicates PII and creates consistency/retention risk.
- **Per-instance percentiles:** cannot be combined correctly; histogram buckets can.
- **LLM-as-judge in extraction:** adds cost, latency, and nondeterminism; semantic evaluation belongs in a sampled asynchronous lane.
- **Immediate self-hosting:** insufficient utilization/quality evidence and high opportunity cost.
- **Exactly-once claims without atomic sink/checkpoint:** misleading; at-least-once is stated and monitored.

### Open decisions

- **Finance/leadership:** Is the post-17-November run-rate approved, and what caused it?
- **Legal/DPO:** Confirm ZDR scope and PII trace retention.
- **Product:** Which routes may downgrade, and under which quality gate?
- **Security:** Approve destructive tool classes, scopes, and operation budgets.
- **Platform:** Confirm API limits, Datadog temporality, window/grace SLO, and whether a pre-export lease is required.

**Evidence:** [`docs/DATA_ANALYSIS.md`](docs/DATA_ANALYSIS.md) · **Engineering depth:** [`docs/DESIGN_APPENDIX.md`](docs/DESIGN_APPENDIX.md) · **Target state:** [`docs/architecture.md`](docs/architecture.md)
