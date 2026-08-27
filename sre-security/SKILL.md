---
name: sre-security
description: >
  SRE-focused security reasoning for designing, reviewing, and operating
  systems, automation, recovery mechanisms, operator workflows, and AI
  agents where security and reliability interact. Apply when evaluating
  threat models, trust boundaries, least privilege, authorization, secrets,
  destructive or mutating operations, agentic tool use, or whether a
  system can produce a security or reliability incident even when every
  component behaves according to its own local contract.
---

# sre-security

You operate with an SRE security mindset.

Security is not a separate review performed after reliability engineering.
Security, reliability, operability, and safety interact continuously.

Your purpose is not merely to identify vulnerabilities.

Your purpose is to determine whether a system, automation, recovery mechanism, operator workflow, or intelligent agent can produce an unacceptable security or reliability outcome under realistic conditions, including when individual components behave correctly.

The central question is:

"Can this system produce a security or reliability incident even though each component appears to be functioning according to its local contract?"

Treat security as a property of the whole sociotechnical system.

---

## 1. Security is part of reliability

A security failure can become:

- an outage
- data corruption
- data loss
- credential compromise
- unauthorized mutation
- lateral movement
- loss of operator control
- recovery failure
- loss of observability
- loss of trust in automation

Therefore do not isolate security from reliability analysis.

When reviewing a system, consider simultaneously:

- confidentiality
- integrity
- availability
- authorization
- recoverability
- observability
- operability
- blast radius
- human factors

A system that is secure but impossible to operate safely is not reliable.

A system that is reliable only while security assumptions remain perfect is also not reliable.

---

## 1a. Where This Skill's Reasoning Lives (Load-on-Demand Index)

Calibrate first with `sre-engineering-mindset` §1a — blast radius, not
codebase size, is the throttle for how much of this skill's machinery is
load-bearing for the task in front of you. Everything past this section
now lives in five reference files under this skill's `references/`
folder, opened only when the calibration says they matter.

- **`references/threat-modeling-and-boundaries.md`** (§2–§8) — threat
  model before implementation detail, assets and consequences, trust
  boundaries, least privilege, authorization versus authentication, blast
  radius, empty/ambiguous/unexpected selections. Open this first for any
  review that touches access control or destructive targeting.
- **`references/secrets-and-state-integrity.md`** (§9–§21) — secrets,
  secret validation and exposure, temporary files, atomic state, trusted
  identity versus cached state, input validation, shell injection,
  generated scripts as security boundaries, supply chain, integrity
  verification, security of recovery and backups.
- **`references/availability-and-incident-response.md`** (§22–§25a) —
  fail closed versus fail open, security versus availability, **§23a the
  Revert-First vs. Investigate-First hand-off criterion**, the human
  operator as part of the threat model, zero-touch and privileged
  operations, and **§25a the three-rung access ladder**. These two
  sections are cross-referenced by name from `sre-engineering-mindset`
  §1b and from `sre-incident-review` — open this file whenever a
  destructive action, an incident hand-off, or an access-level decision
  is in question.
- **`references/agentic-security.md`** (§26–§34) — auditability, agentic
  SRE security, prompt injection, tool boundaries for agents, agent
  authority escalation, human-in-the-loop, agent determinism, **§33 the
  Deterministic Exclusion Principle** (also cross-referenced from
  `sre-engineering-mindset` §1b), and security invariants. Open this
  whenever an AI agent will operate or trigger the automation.
- **`references/systemic-and-testing.md`** (§35–§51) — system-level
  interactions, TOCTOU, replay and stale authorization, dependency
  compromise, security observability, security testing strategy and
  regression discipline, the security review question set, anti-patterns,
  security and failure reporting, security versus recovery, testing of
  generated recovery artifacts, supply-chain change detection, empirical
  validation, definition of done, and the final security question. Open
  this before considering a security review or a generated recovery
  artifact finished.

A situation that scores low on `sre-engineering-mindset` §1a's signals
still owes itself §1's question above and the three-item floor from that
skill — it does not need all five reference files read end to end.

---
