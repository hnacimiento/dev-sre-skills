---
name: sre-incident-review
description: >
  SRE-focused reasoning for analyzing incidents, writing postmortems, and
  performing root-cause and causal analysis. Apply whenever reconstructing
  what happened during a degraded or lost reliability event, evaluating
  evidence and timelines, assessing human and agent decisions during an
  incident, defining corrective actions, or converting incident learning
  into tests, documentation, and system improvement. Treats the incident
  as a sociotechnical system problem, not merely a software defect.
---

# SRE Incident Review

## Purpose

This skill defines how to analyze incidents from a Site Reliability Engineering perspective.

The goal is not to produce an elegant story about what happened or to identify someone to blame.

The goal is to reconstruct the sociotechnical system during the incident, understand how and why the system reached a state of degraded or lost reliability, identify which controls existed, which worked, which failed, and which were insufficient, and transform the resulting knowledge into verifiable changes that reduce recurrence risk without destroying the adaptive capacity of operators.

An incident must not be analyzed only as:

    "Which line of code failed?"

Instead ask:

    "What combination of conditions allowed the system to reach this state,
     why was it not prevented or detected earlier, how was it recovered,
     and what can we learn to improve the complete system?"

The system includes:

- software
- infrastructure
- automation
- AI agents
- observability
- processes
- documentation
- permissions
- interfaces
- operators
- teams
- incentives
- knowledge
- constraints
- procedures
- operational context
- dependencies
- interactions between components
- interactions between humans and automation

This skill must treat the incident as a sociotechnical system problem, not merely as a software defect.

---

# Core Philosophy

## 1. Blameless does not mean causeless

Incident analysis must be blameless.

This does not mean eliminating causality, technical accountability, decision analysis, or ownership.

Do not attribute system failure to personal incompetence when the behavior can be explained by system conditions.

For important operator decisions, analyze:

- what decision was made
- what information was available
- what signals were visible
- what signals were missing
- what alternatives existed
- what constraints existed
- what expectations existed
- what the operator believed was happening
- what the operator did not know
- what tools were available
- what behavior the operator expected from the system
- what feedback the system provided
- why the chosen action made sense under the circumstances

Prefer:

    "Why was this action reasonable given the information and
     constraints available at the time?"

over:

    "Why did someone do something so stupid?"

If an explanation ends with:

    "human error"

continue investigating the conditions that made the error possible, likely, or reasonable.

A conclusion that depends entirely on incompetence, negligence, carelessness, or stupidity should be treated as evidence that the analysis is incomplete.

Blameless analysis does not remove ownership.

Ownership answers who is responsible for improving and operating the system.

Blamelessness answers how causal analysis should be performed.

---

## 1a. Where the Rest of This Reasoning Lives (Load-on-Demand Index)

Calibrate first with `sre-engineering-mindset` §1a — blast radius and how
consequential the incident was, not its length, decide how much of this
skill's machinery is load-bearing. Everything past this section now
lives in five reference files under this skill's `references/` folder,
opened only when the calibration says they matter.

- **`references/evidence-and-reconstruction.md`** (§2–§7a) — facts
  before theories, the evidence hierarchy, the evidence ledger,
  reconstructing before explaining, the timeline as primary evidence,
  preserving uncertainty, and **§7a when the incident was or might have
  been a security incident** (cross-referenced from `sre-security`
  §23a). Open this before writing any narrative about what happened.
- **`references/causal-and-systemic-analysis.md`** (§8–§17) — avoiding a
  single root cause, distinguishing causal levels, why a trigger is not
  a system explanation, analyzing control loops, why local correctness
  is not system safety, work-as-imagined vs. work-as-done, human
  decision context, automation bias, how automation changes where
  complexity lives, and clumsy automation. Open this before naming a
  root cause or a human-error contributor.
- **`references/agentic-and-observability.md`** (§18–§29) — agents as
  part of the system, separating reasoning from authority, deterministic
  boundaries, agent incident analysis, agent security as part of
  incident analysis, detection as part of reliability, detection vs.
  diagnosis vs. recovery confirmation, recovery as a first-class system
  property, demonstrating recovery success, impact before blame, blast
  radius, and careful use of counterfactuals. Open this whenever an
  agent participated in the incident or its response, or detection,
  recovery, or severity need analysis.
- **`references/corrective-actions-and-testing.md`** (§30–§38) —
  avoiding generic action items, action quality, preferring structural
  fixes, converting learning into executable evidence, testing the
  failure mode and not only the fix, why a postmortem is not a
  chronology dump, not optimizing for narrative cleanliness, and **the
  Postmortem Nutrition Loop**. Open this before finalizing action items
  or the postmortem document itself.
- **`references/ai-loop-and-review-procedure.md`** (§39–§46, Final
  Principles, Meta-Rule) — agents not learning from incidents blindly,
  evaluating agent changes, preserving ownership, common anti-patterns,
  the standard review sequence, output expectations, confidence tracking
  evidence, what evidence resolves UNKNOWN, final principles, and the
  meta-rule. Open this when running the standard review sequence or
  deciding what an agent may learn from this incident.

An incident that scores low on `sre-engineering-mindset` §1a's signals
still owes itself that skill's three-item floor — it does not need all
five reference files read end to end.

---
