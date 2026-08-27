# sre-observability — Reference: Review, Drift, and Final Principles (sections 59-67)

This file is loaded on demand from `sre-observability/SKILL.md` §2a, before considering an observability review or design finished. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-observability's own core has
already been read.

---

# 59. Agent Drift and Observability

If an agent participates in production, monitor the operational behavior of the
agent itself.

Examples:

- tool selection patterns
- authorization denials
- verification failures
- action reversal rate
- human rejection rate
- unexpected tool sequences
- policy violations
- prompt injection detections
- stale-context incidents
- escalation frequency

Do not reduce agent health to:

    latency
    token usage
    uptime

The relevant question is:

> Is the agent behaving safely and usefully within its operational contract?

---

# 60. Agent Observability Must Not Depend on Agent Honesty

Never rely solely on:

    agent says it executed X

Use independent evidence:

    tool audit
    API logs
    system state
    resource inspection
    deterministic verification

The system should be capable of disproving the agent's claim.

---

# 61. Observability Hierarchy

When evidence conflicts, prefer stronger evidence.

A useful hierarchy is:

    declared intent
        <
    action request
        <
    execution result
        <
    observed state
        <
    independently verified postcondition

For critical operations, prefer the strongest available evidence.

---

# 62. Do Not Overfit to Happy Paths

For every important success state, define how observability behaves when:

- action succeeds
- action partially succeeds
- action fails
- action is not attempted
- verification fails
- verification is unavailable
- state changes concurrently
- process dies
- telemetry fails

A telemetry design that only describes success is incomplete.

---

# 63. Review Procedure

When reviewing code, documentation, or architecture, perform this sequence.

### Step 1 — Identify operational actions

What can mutate state?

### Step 2 — Identify expected postconditions

What should be true afterward?

### Step 3 — Identify evidence

How can that state be independently observed?

### Step 4 — Identify uncertainty

What can remain unknown?

### Step 5 — Identify partial states

What happens if only some actions succeed?

### Step 6 — Identify recovery

How is recovery observed and verified?

### Step 7 — Identify operator needs

What must a human know during an incident?

### Step 8 — Identify automation/agent behavior

What decisions and actions need auditability?

### Step 9 — Identify telemetry failure

What happens if observability itself fails?

### Step 10 — Test

Inject failures and determine whether the system remains diagnosable.

---

# 64. Severity Heuristic

Use operational impact, not telemetry aesthetics.

### P0

Observability failure can hide or falsely report a condition capable of
causing catastrophic impact.

Examples:

- destructive operation reports SUCCESS while target is wrong
- security mutation occurs without auditable evidence
- recovery reports SUCCESS while system remains corrupted

### P1

Important operational state is hidden, ambiguous, or falsely represented,
requiring significant human intervention or potentially delaying recovery.

### P2

Diagnosis is materially harder or slower, but reliable recovery remains
possible.

### P3

Low-impact telemetry quality issue, documentation gap, or convenience issue.

Severity can change depending on blast radius and context.

---

# 65. Anti-Patterns

Reject or challenge:

### "We have logs, therefore it is observable."

Not necessarily.

### "HTTP 200 means success."

Not necessarily.

### "exit 0 means success."

Only according to the explicit contract.

### "The rollback ran, therefore recovery happened."

False.

### "The agent reported success."

Not evidence.

### "The dashboard is green."

Only as trustworthy as its signals and freshness.

### "More telemetry is always better."

No. It can increase cost and cognitive load.

### "We can reconstruct everything from timestamps."

Not reliably in concurrent distributed systems.

### "Debug logs solve observability."

They may create security and cost problems.

### "If verification fails, the mutation definitely failed."

Not necessarily. The postcondition may simply be unknown.

### "All components are healthy, so the system is healthy."

Not necessarily. System-level interactions can fail.

### "The underlying command/API already guarantees correctness, so verifying it again just adds latency and failure points."

This is a more sophisticated version of "exit 0 means success," and it is
worth naming separately because it sounds like an efficiency argument
rather than a shortcut. Two things are being conflated: what the
underlying primitive actually guarantees, and what could go wrong between
the primitive returning and the state you actually care about. `docker
cp` returning 0 guarantees the copy syscall sequence it performed
succeeded; it does not guarantee the destination file matches the
intended content, that nothing else modified it a moment later, or that
the container you copied into is the one you meant. A verification step
is not a redundant repetition of the same check — it is checking a
different, more specific claim. The realistic cost of an extra check
(milliseconds, one more thing that can time out) should be weighed
against the realistic cost of the failure it catches (silently corrupted
state reaching production, discovered later, with no evidence of when or
how). A verification step that fails is not "one more failure point" in a
bad sense — it converts a silent, undetected corruption into a visible,
diagnosable one, which is close to the whole purpose of this skill.

---

# 66. Required Review Questions

When reviewing a system, explicitly ask:

1. What does success actually mean?
2. What evidence proves success?
3. Can the system distinguish success from attempted success?
4. Can it distinguish failure from unknown?
5. Can it expose partial completion?
6. Can it reconstruct state transitions?
7. Can it reconstruct causality?
8. Can it identify the affected resources?
9. Can it prove recovery?
10. Can telemetry itself fail?
11. What happens if telemetry is unavailable?
12. Can logs expose secrets?
13. Can stale telemetry mislead the operator?
14. Can dashboards hide integrity problems?
15. Can an operator understand the state during an incident?
16. Can an agent's claims be independently verified?
17. Can external content manipulate the agent?
18. Are destructive operations independently auditable?
19. Are important invariants observable?
20. What evidence survives process death?
21. What evidence survives an outage?
22. What evidence is retained for postmortem analysis?
23. Can the telemetry distinguish intended state from actual state?
24. Can the system demonstrate what it does not know?

---

# 67. Final SRE Rule

The strongest observability question is not:

> "What telemetry do we have?"

It is:

> "If this system failed right now in a way we did not anticipate, could an
> engineer determine what happened, what state remains, what is uncertain,
> what is safe to do next, and later reconstruct why the system behaved that
> way?"

If the answer is no, improve the system's observability before assuming the
problem is merely a missing dashboard, metric, or log.

Observability is not decoration around reliability.

It is part of the mechanism by which humans and automation maintain control of
the system.