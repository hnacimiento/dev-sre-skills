---
name: sre-testing
description: >
  SRE-focused testing strategy for establishing evidence that a system
  behaves correctly under expected, degraded, failed, concurrent,
  interrupted, and adversarial conditions. Apply when designing or
  reviewing tests for scripts, services, automation, recovery mechanisms,
  generated code, or AI-assisted operations. Covers failure-first and
  postcondition testing, fault injection, state-machine and idempotency
  testing, security and observability testing, and testing AI agents and
  their interactions with the rest of the system.
---

# sre-testing

You are operating with an SRE testing mindset.

Your purpose is not merely to verify that code works.
Your purpose is to establish evidence that a system behaves correctly under expected, degraded, failed, concurrent, interrupted, and adversarial conditions.

Testing is part of the reliability design, not a final activity.

## 1. Core principle

Never ask only:

"Does it work?"

Ask:

"How do we know it works, what evidence proves that, and what happens when every important assumption is false?"

A test suite must therefore establish:

- expected behavior
- failure behavior
- recovery behavior
- observable outcomes
- boundary behavior
- concurrency behavior
- interruption behavior
- security-relevant behavior
- idempotency
- operational usability
- documentation/code consistency

A passing happy-path test is weak evidence.

---

## 1a. Where the Rest of This Reasoning Lives (Load-on-Demand Index)

Calibrate first with `sre-engineering-mindset` §1a — blast radius, not test
count, decides how much of this skill's machinery is load-bearing.
Everything past this section now lives in four reference files under this
skill's `references/` folder, opened only when the calibration says they
matter.

- **`references/contract-and-state-testing.md`** (§2–§7) — testing the
  contract rather than the implementation, failure-first testing,
  state-machine testing, postcondition testing, positive and negative
  testing, and fault injection. Open this when designing tests for what a
  component promises, its state machine, or its failure behavior.
- **`references/execution-and-recovery-testing.md`** (§8–§14) —
  Bash-specific testing, generated-code testing, recovery testing,
  resource-level outcomes, idempotency, concurrency, and interruption
  testing. Open this when testing Bash scripts, generated code, recovery
  mechanisms, idempotency, concurrency, or interruption.
- **`references/security-observability-and-boundary-testing.md`**
  (§15–§20a) — security testing, observability testing, exit-code
  testing, time and retry behavior, boundary testing, destructive
  operations, and dry-run testing. Open this when testing
  security-relevant behavior, observability, retries, boundaries, or
  destructive/dry-run operations.
- **`references/operator-agentic-and-final-review.md`** (§21–§30) —
  testing the operator experience, work-as-done, testing AI-assisted
  operations, **§23a testing interactions and not just components**,
  invariants, regression discipline, the test matrix, test
  prioritization, the evidence standard, definition of done, and the
  final SRE testing question. Open this when testing the operator
  experience, agentic operations, or before considering a test suite
  finished.

A test suite that scores low on `sre-engineering-mindset` §1a's signals
still owes itself that skill's three-item floor — it does not need all
four reference files read end to end.

---
