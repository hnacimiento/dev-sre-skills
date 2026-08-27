# sre-incident-review — Reference: Corrective Actions, Testing, and the Postmortem Nutrition Loop (sections 30-38)

This file is loaded on demand from `sre-incident-review/SKILL.md`'s
load-on-demand index, before finalizing action items or the postmortem document itself. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-incident-review's own core has already been read.

---

# Corrective Actions

## 30. Avoid generic action items

Avoid:

    "Improve monitoring."

    "Add more tests."

    "Improve documentation."

    "Train the team."

    "Be more careful."

A corrective action should be:

- specific
- assignable
- observable
- verifiable
- tied to a failure mechanism
- tied to a measurable risk reduction

---

## 31. Action quality

For every important corrective action ask:

1. What risk does it reduce?
2. What failure mechanism does it address?
3. What system property changes?
4. How will implementation be verified?
5. How will risk reduction be verified?
6. What new risk does it introduce?
7. Does it increase or decrease toil?
8. Does it increase or decrease complexity?
9. Does it affect recovery?
10. Does it require failure injection?
11. Can the original failure mode still occur?
12. If it can still occur, will it now be detected or contained?
13. How will we know the control remains effective later?

---

## 32. Prefer structural fixes

When appropriate, prioritize:

1. eliminate the possibility of the failure
2. reduce blast radius
3. add prevention
4. add containment
5. improve detection
6. improve recovery
7. improve observability
8. improve tooling
9. improve documentation
10. training

Training should not be the only control for a systemic failure that can be prevented technically.

Do not automatically prefer prevention over recovery when prevention would create excessive complexity or reduce adaptive capacity.

Evaluate the complete control strategy.

---

# Testing After Incidents

## 33. Convert important learning into executable evidence

When possible, convert incident learning into:

- tests
- invariants
- alerts
- validations
- policies
- simulations
- fault injection
- GameDays
- canary conditions
- recovery drills
- automated regression tests
- agent evaluations

A lesson that exists only in prose can be forgotten or bypassed.

---

## 34. Test the failure mode, not only the fix

Do not stop at:

    "The new code works."

Prefer demonstrating:

    "The failure mode that produced the incident can no longer
     produce the same unsafe state, or is now detected, contained,
     or recovered safely."

Test:

- the original failure mode
- partial failure
- timeout
- network loss
- stale state
- concurrent operations
- corrupted state
- interrupted execution
- process termination
- SIGKILL where relevant
- dependency failure
- unexpected input
- recovery verification failure

---

# Postmortem Quality

## 35. A postmortem is not a chronology dump

A useful postmortem should answer:

- what happened
- when it happened
- what impact occurred
- how it was detected
- what operators knew at each point
- what hypotheses existed
- what actions were taken
- what worked
- what failed
- how recovery occurred
- how recovery was verified
- why existing controls did not prevent or contain the impact
- what contributing conditions existed
- what remains UNKNOWN
- what will change
- how the changes will be validated

---

## 36. Do not optimize for narrative cleanliness

Operational reality may be:

- confusing
- simultaneous
- contradictory
- incomplete
- adaptive
- nonlinear

Do not transform it into an artificial linear story.

If multiple hypotheses existed simultaneously, preserve them.

If actions that appear incorrect retrospectively were reasonable given the information available at the time, explain that context.

A clean narrative is not necessarily an accurate narrative.

---

# Postmortem Nutrition Loop

## 37. Incidents must improve the future system

A postmortem does not end when it is published.

Learning should feed:

- code
- tests
- observability
- automation
- runbooks
- documentation
- policies
- training
- GameDays
- architecture
- agent evaluations
- agent policies
- operational tooling

Conceptually:

    INCIDENT
        ↓
    EVIDENCE
        ↓
    HUMAN ANALYSIS
        ↓
    POSTMORTEM
        ↓
    SYSTEM CHANGES
        ↓
    TEST / VALIDATION
        ↓
    FUTURE OPERATIONS
        ↓
    NEW EVIDENCE
        ↓
    NEXT INCIDENT / LEARNING CYCLE

The loop should remain active.

---

## 38. Human-generated operational knowledge matters

Do not eliminate the human learning component.

Human timelines and decisions may contain information not visible in telemetry:

- hypotheses
- ambiguous signals
- ignored signals
- mental models
- trade-offs
- discoveries
- workarounds
- constraints
- contextual reasoning

This information can benefit:

- future operators
- documentation
- automation
- tooling
- agent evaluation
- training
- simulation
- system redesign

The postmortem is not merely documentation.

It is a mechanism for preserving operational knowledge.

This is also where automating incident response too aggressively creates a slow, easy-to-miss failure: as operators intervene less often, they have less first-hand struggle to draw on when they do write a postmortem, so postmortems get thinner — fewer discarded hypotheses, less hard-won context, less of the texture that makes a postmortem useful training material rather than a bare timeline. Thinner postmortems are exactly the input the loop above and the agent-evaluation loop below both depend on. The visible symptom to watch for is an agent that starts recommending actions which used to work but no longer fit the system as it has evolved — that is the lagging indicator that the human side of this loop has gone quiet, not evidence that the agent itself has regressed. (The same mechanism is documented from the observability angle in sre-observability; treat it as one phenomenon described from two skills, not two separate risks.) (sre-release-deployment carries a third version of this same loop,
adapted to release engineering specifically. Maintenance note: when
revising the underlying concept, check all three copies rather than
assuming this one is the only one.)

---

