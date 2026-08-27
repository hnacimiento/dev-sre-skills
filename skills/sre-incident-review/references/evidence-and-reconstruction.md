# sre-incident-review — Reference: Evidence Discipline and Reconstruction (sections 2-7a)

This file is loaded on demand from `sre-incident-review/SKILL.md`'s
load-on-demand index, before writing any narrative about what happened, and whenever the incident may have a security dimension (see 7a). It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-incident-review's own core has already been read.

---

# Evidence Discipline

## 2. Facts before theories

Explicitly distinguish:

- observed facts
- evidence
- interpretations
- hypotheses
- conclusions
- uncertainties

Never present a hypothesis as a fact.

Never silently fill gaps in the evidence.

When evidence is insufficient, explicitly state:

    UNKNOWN

or:

    "There is insufficient evidence to determine this."

Do not invent:

- timestamps
- operator intent
- unavailable telemetry
- missing commands
- missing system states
- causal relationships
- successful recovery
- expected behavior
- historical context that was not observed

A partially reconstructed timeline is preferable to an invented complete timeline.

---

## 3. Evidence hierarchy

Prefer direct operational evidence.

A useful evidence hierarchy is:

1. metrics and events recorded during the incident
2. logs
3. traces
4. persisted system state
5. recorded configuration or deployment changes
6. audited commands and actions
7. tool execution timelines
8. communications contemporaneous with the incident
9. operator statements collected after the incident
10. human memory and retrospective reconstruction

This hierarchy is contextual, not absolute.

Lower-ranked evidence is not automatically false.

However, retrospective memory should not silently override contemporaneous evidence.

When sources conflict:

- preserve the conflict
- identify the conflicting observations
- do not silently choose one
- state which evidence is stronger and why, if possible
- mark unresolved conclusions as UNKNOWN

---

## 4. Evidence ledger

For complex incidents, maintain an explicit or mental evidence ledger:

EVIDENCE
- What was directly observed?

INTERPRETATION
- What does the evidence appear to mean?

HYPOTHESIS
- What could explain the observation?

CONFIDENCE
- high / medium / low

UNKNOWN
- What cannot currently be determined?

This prevents:

- correlation being mistaken for causation
- temporal proximity being mistaken for mechanism
- assumptions being promoted into facts
- retrospective knowledge being projected backward onto operators

---

# Reconstruction Before Explanation

## 5. Reconstruct before explaining

Before explaining why an incident happened, reconstruct what happened.

Attempt to establish:

1. normal state before the incident
2. first observable deviation
3. detection
4. initial interpretation
5. actions taken
6. observable system changes
7. escalation
8. mitigation
9. recovery
10. stabilization
11. post-recovery state
12. subsequent learning

Do not construct the timeline retrospectively as if operators knew the eventual cause from the beginning.

Preserve the uncertainty that existed at each point in time.

---

## 6. Timeline is primary evidence

The timeline is a central postmortem artifact.

Do not reduce it to:

    10:05 incident begins
    10:20 team investigates
    10:45 rollback
    11:00 recovery

When possible record:

- timestamp
- event
- signal available at the time
- interpretation at the time
- action taken
- observable result
- hypothesis change
- uncertainty

For example:

    10:05

    Event:
    error rate increases

    Signal available:
    alert A

    Interpretation:
    possible capacity saturation

    Action:
    increase capacity

    Observable result:
    error rate continues increasing

    New information:
    metric B begins degrading

    Hypothesis change:
    capacity saturation becomes less likely;
    dependency failure becomes more plausible

The objective is to reconstruct the evolution of the operators' mental model.

---

## 7. Preserve uncertainty

Never write:

    "The operator knew X was the cause."

unless evidence demonstrates that this was actually known at that moment.

Prefer:

    "At that time, X, Y, and Z were plausible hypotheses.
     Available information did not allow them to be distinguished."

The incident must preserve uncertainty as part of the operational context.

Do not use hindsight to make past decisions appear obviously incorrect.

---

### 7a. When the Incident Was (or Might Have Been) a Security Incident

Everything above in this Evidence Discipline section assumes ordinary
operational failure, where reconstructing the timeline from logs,
metrics, and operator recollection is safe to do after the fact and
after the system has already been mutated back toward a working state.
That assumption breaks if the incident was, or might have been, an
active security intrusion rather than a software or human error — see
sre-security §23a for the signals that should raise this suspicion and
the criterion for transferring incident command to a Security Incident
Commander.

If that hand-off happened (or should have, in hindsight), the postmortem
for this incident inherits different evidence-discipline requirements
than the rest of this skill assumes:

- the timeline (§6) must be reconstructed from evidence preserved
  *before* any containment mutation, not from state observed afterward —
  restarts, reverts, and cleanup performed during containment can
  destroy exactly the evidence a causal analysis needs;
- the evidence ledger (§4) should record explicitly what was preserved
  before containment and what was necessarily altered by it, so a later
  reader does not mistake a containment side-effect for a natural
  consequence of the original failure;
- "preserve uncertainty" above still applies, with one addition: do not
  assume the absence of a smoking gun means no intrusion occurred —
  the evasive-behavior signals in sre-security §23a exist precisely
  because a competent attacker minimizes the evidence they leave.

This does not turn every incident review into a security review. It
means the evidence-discipline questions above should include, early,
"was this handed off to Security, and if not, should it have been?" —
answered honestly even when the answer is uncomfortable in hindsight.

---

