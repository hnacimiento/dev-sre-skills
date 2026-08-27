# sre-observability — Reference: Operations, Toil, and Generated Artifacts (sections 43-58)

This file is loaded on demand from `sre-observability/SKILL.md` §2a, whenever dashboards, incident mode, generated artifacts, or the Postmortem Nutrition Loop are in question. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-observability's own core has
already been read.

---

# 43. Dashboards

Dashboards should answer operational questions.

A useful dashboard should help answer:

- Are users affected?
- What changed?
- Where is the problem?
- What dependencies are failing?
- What recovery is running?
- What remains broken?
- What is uncertain?
- What action is safe next?

Avoid dashboards that merely display every available metric.

---

# 44. Incident Mode

Observability requirements change during incidents.

During an incident, prioritize:

1. user impact
2. current system state
3. recent changes
4. active failures
5. dependency state
6. ongoing mutations
7. recovery state
8. uncertainty
9. safe next actions

Do not bury critical state beneath debug noise.

Incident interfaces should reduce cognitive load.

---

# 45. Human Factors

Observability is part of the operator interface.

Ask:

- Can an operator understand this at 03:00?
- Are failures distinguishable?
- Are partial states visible?
- Is uncertainty explicit?
- Can the operator determine what happened without reading thousands
  of lines?
- Does the interface encourage unsafe actions?
- Does the tool explain what will happen before a dangerous mutation?
- Does the system expose enough state to maintain an accurate mental model?

A technically complete telemetry system can still be operationally unusable.

---

# 46. Avoid Observability Theater

Do not add telemetry merely because:

- dashboards look professional
- logs are required by convention
- metrics exist in a template
- every function prints something
- "more visibility" sounds good

Every important signal should have an operational consumer.

Ask:

> "What decision will this signal change?"

If the answer is none, reconsider it.

---

# 47. Observability and Toil

Poor observability creates toil.

Examples:

- operator must grep thousands of lines
- operator manually correlates timestamps
- operator must inspect containers one by one
- operator must infer state from shell output
- operator must parse human prose to determine success
- operator must repeatedly run commands to verify state

Good observability should make routine diagnosis increasingly automatic.

But do not automate away the evidence needed for human understanding.

---

# 48. Observability and Automation

Automation should expose:

- what it intends to do
- what it did
- what succeeded
- what failed
- what was skipped
- why it skipped it
- what remains unknown
- what it expects the operator to do

Avoid opaque:

    "Automation failed."

Prefer:

    operation=rollback
    target=container-42
    resource=addon-3.js
    action=restore
    result=ATTEMPT_FAILED
    reason=hash_mismatch
    expected=<hash>
    observed=<hash>
    recovery_status=PARTIAL
    operator_action=required

Do not expose secrets while doing so.

---

# 49. Observability of Generated Artifacts

If a program generates executable artifacts, observability must extend across
the generation boundary.

Record where appropriate:

- generation time
- source operation ID
- artifact type
- artifact version
- validation result
- checksum
- intended target
- dependencies
- recovery relationship

A generated rollback script is not merely a file.

It is a future operational control surface.

---

# 50. Observability Contracts

For important operations, define an explicit observability contract.

The contract should state:

### Inputs

What must be known before the operation?

### Action

What mutation is performed?

### Expected outcome

What state should exist afterward?

### Evidence

What observations prove that state?

### Failure states

What failure classes exist?

### Unknown states

What cannot currently be determined?

### Recovery

What evidence proves recovery?

### Exit status

How does the external interface summarize the result?

### Audit

What durable record remains?

If any of these are undefined for a critical operation, observability is
incomplete.

---

# 51. Verification Matrix

When reviewing an operation, reason in a matrix:

| Operation | Intended state | Action evidence | Postcondition | Integrity evidence | Result |
|---|---|---|---|---|---|
| Deploy | version X | command/API | version X observed | artifact hash | VERIFIED |
| Restore file | backup Y | copy result | file exists | hash match | VERIFIED |
| Delete | resource absent | delete result | resource absent | identity check | VERIFIED |
| CSS update | config X | POST result | GET returns X | semantic comparison | VERIFIED |
| Agent rollback | target X | tool execution | target X restored | deterministic verification | VERIFIED |

Do not mark an operation successful merely because the "Action evidence" column
is successful.

---

# 52. Failure Injection

Observability should be tested under failure.

Inject:

- command failure
- timeout
- stale state
- corrupted response
- missing telemetry
- delayed telemetry
- duplicate events
- out-of-order events
- process termination
- partial mutation
- failed verification
- unavailable dependency
- authorization failure
- concurrent mutation

Ask:

> Can the operator still determine what actually happened?

If not, the observability design is incomplete.

---

# 53. Test the Telemetry Itself

Telemetry is software.

Test:

- log schema
- event emission
- missing events
- duplicate events
- malformed events
- sensitive-data filtering
- correlation IDs
- timestamp behavior
- retry behavior
- aggregation logic
- alert conditions
- dashboard assumptions

Do not assume telemetry is correct merely because the application is correct.

---

# 54. Observability During Partial Failure

The system must remain diagnosable while degraded.

Ask:

- What happens if the API is down?
- What happens if logging backend is down?
- What happens if metrics backend is unavailable?
- What happens if the container disappears?
- What happens if verification cannot run?
- What happens if the process dies halfway through?

Do not create a circular dependency where:

    system failure
       ↓
    observability failure
       ↓
    no evidence
       ↓
    impossible diagnosis

Critical local evidence may need to survive external telemetry outages.

---

# 55. Observability Failure Must Have Its Own Model

Distinguish:

    SYSTEM_FAILURE

from:

    OBSERVABILITY_FAILURE

and:

    SYSTEM_FAILURE + OBSERVABILITY_FAILURE

The third case is particularly dangerous.

A healthy-looking system with broken telemetry may be less trustworthy than a
clearly failing system.

---

# 56. Recovery Evidence Must Survive Process Death

For critical recovery operations, durable evidence should exist independently
of the process's final exit status.

Examples:

- manifest
- per-resource result
- recovery artifact
- state file
- durable event
- checksum
- audit record

This matters for:

- SIGKILL
- power loss
- kernel panic
- container termination
- host failure

The absence of a final summary must not erase the history needed for recovery.

---

# 57. Postmortem Observability

After an incident, observability should allow reconstruction of:

1. initial condition
2. first signal
3. detection
4. interpretation
5. actions
6. mutations
7. failures
8. recovery
9. verification
10. final state

A postmortem should not require reconstructing reality entirely from memory.

High-quality operational evidence becomes part of the organization's learning
system.

---

# 58. Postmortem Nutrition Loop

Treat incident evidence as feedback into engineering.

    INCIDENT
       ↓
    OBSERVATIONS
       ↓
    HUMAN INTERPRETATION
       ↓
    POSTMORTEM
       ↓
    SYSTEM / TOOL / PLAYBOOK CHANGES
       ↓
    FUTURE OBSERVABILITY

For AI-assisted operations, this loop may additionally feed:

    POSTMORTEM
       ↓
    EVALUATION DATA
       ↓
    AGENT TESTS / POLICIES / TOOLS

Do not automatically feed raw incident data into an agent or model.

Preserve:

- provenance
- review
- security
- privacy
- correctness
- context

These two failure modes are not independent; they reinforce each other.
As automation and agents absorb more incident response, operators
practice diagnosis less often (see sre-engineering-mindset, cognitive
atrophy), which means the postmortems they write become thinner and less
insightful — there is less first-hand struggle, fewer discarded
hypotheses, less hard-won context to record. Thinner postmortems are
exactly the input this loop depends on to keep agent playbooks and
evaluation data current. The result is a slow joint decline: humans lose
practice at the same time as the material meant to keep automation
current dries up, and neither failure is very visible on its own week to
week. Treat sustained automation of incident response as something that
needs a deliberate countermeasure to keep this loop fed — for example,
preserving deliberate human-led investigation on a sampled subset of
incidents specifically to keep postmortem quality, and therefore agent
evaluation quality, from eroding quietly.

(Maintenance note: sre-incident-review and sre-release-deployment each
carry their own version of this loop, adapted to their own domain — when
revising the underlying concept here, check those two as well rather than
assuming this copy is the only one.)

---

