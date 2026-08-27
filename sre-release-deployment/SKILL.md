---
name: sre-release-deployment
description: >
  SRE-focused reasoning for software releases, deployments, rollouts,
  configuration changes, database migrations, feature flags, and other
  production mutations. Apply when preparing, reviewing, or operating a
  change to a running system — application deployments, CI/CD pipelines,
  container or Kubernetes releases, infrastructure changes, schema
  changes, package/dependency upgrades, generated recovery artifacts,
  automated remediation, or agent-assisted releases — regardless of
  tooling.
---

SKILL.md — SRE Release & Deployment
===================================

# SRE Release & Deployment

## Purpose

Apply a Site Reliability Engineering mindset to software releases,
deployments, rollouts, configuration changes, migrations, infrastructure
changes, and other production mutations.

The central principle is:

> A deployment is not successful because a deployment command succeeded.
> A release is successful only when the resulting production state has been
> verified to be acceptable.

Treat every production change as a controlled change to a living system.

The objective is not merely to move artifacts from A to B.

The objective is to:

- control change risk
- minimize blast radius
- preserve recoverability
- verify intended state
- detect unintended state
- make failure observable
- make partial failure explicit
- prevent unsafe automation
- preserve operator understanding
- maintain accountability
- continuously improve the release process

This skill applies to:

- application deployments
- Bash-based installers
- CI/CD pipelines
- container deployments
- Kubernetes releases
- infrastructure changes
- configuration changes
- database migrations
- schema changes
- feature flags
- package upgrades
- dependency upgrades
- firmware/software updates
- generated recovery artifacts
- automated remediation
- agent-assisted releases
- manual production changes

It applies regardless of tooling.

---

# 1. Core Mental Model

Do not think:

    build -> deploy -> exit 0

Think:

    intent
      ↓
    change preparation
      ↓
    preconditions
      ↓
    controlled mutation
      ↓
    observation
      ↓
    verification
      ↓
    decision
      ↓
    continue / pause / abort / rollback / roll-forward
      ↓
    final-state verification
      ↓
    evidence
      ↓
    learning

A release is a state transition.

The important question is:

> Did the system transition from the expected previous state into the
> intended acceptable state?

Not:

> Did the command return zero?

---

## 1a. Where the Rest of This Reasoning Lives (Load-on-Demand Index)

Calibrate first with `sre-engineering-mindset` §1a — blast radius, not
release size, decides how much of this skill's machinery is load-bearing
for the change in front of you. Everything past this section now lives
in five reference files under this skill's `references/` folder, opened
only when the calibration says they matter.

- **`references/change-contract-and-rollout.md`** (§2–§16) — change as a
  reliability event, the change contract, preconditions (including
  preconditions about the real target), artifact integrity and
  provenance, blast radius, progressive delivery, canary as an
  experiment, deployment verification, technical health vs. system
  health, independent verification, the rollout state machine, partial
  deployment, and why UNKNOWN is not SUCCESS. Open this for any change
  that needs a go/no-go decision.
- **`references/rollback-and-recovery.md`** (§17–§24) — retry safety,
  idempotency, why rollback is not magic, rollback vs. roll-forward,
  verifying recovery, treating recovery as a separate change, database
  migrations, and expand/contract. Open this whenever the change might
  need to be undone or forward-fixed.
- **`references/operational-controls-and-concurrency.md`** (§25–§36) —
  configuration changes, feature flags, change freeze, error budget,
  change failure rate, deployment observability, auditability, secrets,
  concurrency, deployment ordering, why intermediate states matter, and
  destructive changes. Open this whenever the change touches shared
  state, secrets, or has a non-trivial intermediate state.
- **`references/agentic-and-systemic.md`** (§37–§52) — human factors,
  work-as-done, automation vs. toil, agentic release operations, why
  agent outputs are not evidence, deterministic guarantees, agent threat
  model, agent tool boundaries, STPA / system-level thinking, safety
  invariants, failure injection, `kill -9` and process death, generated
  recovery artifacts, release state surviving process loss, rollback
  artifacts needing to be tested, and release-path supply-chain risk
  (compromised dependencies, build systems, or CI). Open this whenever an
  AI agent triggers or assists the release, or the blast radius is high
  enough that component correctness cannot be trusted to imply
  system-level safety.
- **`references/documentation-postmortems-and-review.md`** (§53–§64) —
  documentation as an operational interface, release runbooks, incident
  interaction, the postmortem feedback loop, **the Postmortem Nutrition
  Loop**, why release metrics must not become targets, anti-patterns
  (including **§59a the Checklist Trap**), the decision framework, the
  required reasoning pattern, review severity and style, and the final
  principle. Open this before considering a release process, runbook, or
  review finished.

A change that scores low on `sre-engineering-mindset` §1a's signals still
owes itself that skill's three-item floor — it does not need all five
reference files read end to end.

---
