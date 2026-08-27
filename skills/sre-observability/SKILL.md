---
name: sre-observability
description: >
  Applies an SRE observability mindset to systems, automation, deployments,
  recovery procedures, scripts, services, and AI-assisted operations. Focuses
  on whether the system exposes enough trustworthy evidence to determine what
  actually happened, what state it is in, why it is there, and what an operator
  can safely do next. Treats observability as a property of the system and its
  operational control loop, not as a collection of dashboards, metrics, logs,
  or traces.
---

# SRE Observability Mindset

## 1. Purpose

When reviewing, designing, implementing, debugging, or documenting a system,
reason as an SRE responsible for operating that system under both normal and
abnormal conditions.

The objective is not:

> "Do we have logs, metrics, and dashboards?"

The objective is:

> "Can an engineer determine what the system actually did, what state it is
> actually in, what evidence supports that conclusion, what remains uncertain,
> and what action is safe to take next?"

Observability exists to support a reliable operational control loop:

    SYSTEM
       ↓
    SIGNALS
       ↓
    INTERPRETATION
       ↓
    DECISION
       ↓
    ACTION
       ↓
    VERIFICATION
       ↓
    SYSTEM

A system that performs actions but cannot reliably expose their outcome is
operationally incomplete.

A system that emits enormous amounts of telemetry but cannot answer operational
questions is not necessarily observable.

---

# 2. Core Principle

## Evidence over assertion

Never equate:

- command succeeded
- function returned 0
- HTTP returned 2xx
- log says SUCCESS
- metric is green
- process is alive
- deployment completed
- rollback ran
- agent said it succeeded

with:

> the desired state was achieved.

Success must be supported by evidence appropriate to the operation.

Prefer:

    ACTION
      ↓
    EXPECTED POSTCONDITION
      ↓
    OBSERVATION
      ↓
    VERIFIED STATE

over:

    ACTION
      ↓
    exit 0
      ↓
    "SUCCESS"

---

## 2a. Where the Rest of This Reasoning Lives (Load-on-Demand Index)

Calibrate first with `sre-engineering-mindset` §1a — blast radius, not the
amount of telemetry already present, decides how much of this skill's
machinery is load-bearing. Everything past this section now lives in
five reference files under this skill's `references/` folder, opened
only when the calibration says they matter.

- **`references/control-loop-and-state.md`** (§3–§16) — observability as
  a set of questions, the control-loop property, desired vs. observed
  state, never hiding unknown, status semantics, the six telemetry types
  (including **§8a the six related terms that are not synonyms**), logs
  as evidence rather than truth, structured events, not logging secrets,
  why logging level is not a security boundary, correlation and
  causality, observable state transitions, visible partial failure, and
  resource-level observability. Open this whenever you need to decide
  what telemetry a system should emit about its own state.
- **`references/verification-and-signals.md`** (§17–§30) — observable
  recovery, verification as a telemetry producer, exit codes and
  observability, alerts representing user-relevant failure, symptoms vs.
  causes, SLO-oriented observability, why golden signals are not the
  entire model, integrity as observable state, freshness and staleness,
  observability of verification itself, retry semantics, idempotency and
  observability, concurrency, and security observability. Open this
  whenever recovery, alerting, SLOs, or verification telemetry are in
  question.
- **`references/agentic-observability.md`** (§31–§42) — agentic SRE
  observability, why agent output is not evidence, agent decision vs.
  agent action, prompt-injection observability, tool-level observability
  for agents, deterministic verification, system-level observability,
  invariants, missing telemetry as its own signal, cardinality and cost,
  retention and incident reconstruction, and time/clock problems. Open
  this whenever an AI agent's actions, decisions, or outputs need to be
  observable.
- **`references/operations-and-generated-artifacts.md`** (§43–§58) —
  dashboards, incident mode, human factors, avoiding observability
  theater, observability and toil, observability and automation,
  observability of generated artifacts, observability contracts, the
  verification matrix, failure injection, testing the telemetry itself,
  observability during partial failure, observability failure needing
  its own model, recovery evidence surviving process death, postmortem
  observability, and **the Postmortem Nutrition Loop**. Open this
  whenever dashboards, incident mode, generated artifacts, or the
  nutrition loop are in question.
- **`references/review-and-final.md`** (§59–§67) — agent drift and
  observability, why agent observability must not depend on agent
  honesty, the observability hierarchy, not overfitting to happy paths,
  review procedure, severity heuristic, anti-patterns, required review
  questions, and the final SRE rule. Open this before considering an
  observability review or design finished.

A system that scores low on `sre-engineering-mindset` §1a's signals still
owes itself that skill's three-item floor — it does not need all five
reference files read end to end.

---
