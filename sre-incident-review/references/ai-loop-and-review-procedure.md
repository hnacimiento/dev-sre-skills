# sre-incident-review — Reference: AI Learning Loop, Ownership, Anti-Patterns, and Review Procedure (sections 39-46, Final Principles, Meta-Rule)

This file is loaded on demand from `sre-incident-review/SKILL.md`'s
load-on-demand index, when running the standard review sequence or deciding what an agent may learn from this incident. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-incident-review's own core has already been read.

---

# AI Learning Loop

## 39. Agents must not learn from incidents blindly

If postmortems influence AI agents, distinguish:

- verified facts
- human decisions
- hypotheses
- successful actions
- failed actions
- contextual actions
- incident-specific conditions
- generalizable controls

Never convert:

    "This worked once."

automatically into:

    "This should always be done."

Incident actions are contextual evidence, not necessarily universal policies.

---

## 40. Evaluate agent changes

If an incident results in a modification to:

- prompt
- playbook
- policy
- tool
- model
- agent
- permissions
- workflow

verify:

- what behavior changed
- what scenario it is intended to improve
- what assumptions it introduces
- what new scenarios it affects
- what regressions it could introduce
- how it will be evaluated
- what evidence demonstrates improvement

Prefer reproducible evaluations and deterministic boundaries around agents rather than relying only on textual instructions.

---

# Operational Ownership

## 41. Preserve ownership

Incident analysis should establish:

- who operates the system
- who owns the service
- who maintains the code
- who maintains automation
- who maintains alerts
- who maintains runbooks
- who maintains agents
- who validates changes
- who owns recovery procedures
- who owns security boundaries

Automation can execute work.

Agents can reason and recommend.

Neither replaces organizational ownership.

Automation does not eliminate the need for humans to remain connected to production.

Removing toil is not equivalent to removing operational contact.

---

# Common Anti-Patterns

## 42. Detect these explicitly

Treat the following as signals of weak analysis:

- searching for a culprit
- finding one root cause too quickly
- confusing trigger with system explanation
- assuming causality from temporal proximity
- presenting hypotheses as facts
- ignoring UNKNOWN
- using "human error" as the final explanation
- assuming exit 0 means success
- assuming rollback means recovery
- assuming health checks prove complete system health
- adding only more alerts
- adding only more tests
- adding only documentation
- blaming operators for not following impractical procedures
- blaming an agent without analyzing context, tools, permissions, and guardrails
- assuming human-in-the-loop eliminates risk
- assuming deterministic components guarantee global safety
- ignoring controller interactions
- ignoring latent conditions
- ignoring organizational conditions
- closing an incident without testing the failure mode
- marking an action item complete without evidence
- turning a contextual workaround into a universal policy
- using hindsight to judge operator decisions
- treating a clean narrative as proof of understanding
- assuming absence of evidence means absence of a condition
- assuming successful execution means successful state transition

---

# Review Procedure

## 43. Standard incident review sequence

When reviewing an incident, use this sequence as a reasoning guide:

1. Define impact.
2. Collect evidence.
3. Establish normal state.
4. Build the timeline.
5. Separate facts from hypotheses.
6. Preserve uncertainty.
7. Reconstruct operator decision context.
8. Identify the trigger.
9. Identify immediate mechanisms.
10. Identify contributing conditions.
11. Identify controls.
12. Analyze failed and successful controls.
13. Analyze automation.
14. Analyze control loops.
15. Analyze human-system interaction.
16. Analyze AI agents if present.
17. Analyze security implications.
18. Analyze blast radius.
19. Analyze detection.
20. Analyze diagnosis.
21. Analyze recovery.
22. Verify recovery evidence.
23. Identify UNKNOWNs.
24. Formulate system-level explanations.
25. Design corrective actions.
26. Convert important learning into tests or controls.
27. Define validation criteria.
28. Define ownership.
29. Define how learning feeds the next operational cycle.

This sequence is a reasoning aid, not a bureaucratic requirement to produce twenty-nine sections.

The analysis must remain iterative.

If new evidence invalidates an earlier hypothesis:

- update the hypothesis
- preserve the reasoning change
- do not hide the earlier uncertainty

If the investigation needs to return to the timeline, return to it.

---

# Output Expectations

## 44. When asked to review an incident

Prioritize the information that is useful for decision-making.

Depending on context, structure the response around:

### Impact

What happened and who or what was affected.

### Evidence

What is actually known.

### Timeline

How the incident evolved.

### Decision Context

What operators knew and believed at each point.

### System Analysis

What mechanisms and conditions produced the incident.

### Control Analysis

What controls existed and how they interacted.

### Automation / Agent Analysis

How automation or agents participated, if applicable.

### Detection

How the incident was detected and what delayed detection.

### Recovery

How the system recovered and how recovery was demonstrated.

### Unknowns

What cannot currently be determined.

### Contributing Conditions

What conditions allowed or amplified the incident.

### Corrective Actions

What concrete changes should be made.

### Validation

How the changes will be proven effective.

### Learning Loop

How the learning will become part of the operational system.

Do not produce every section when the problem does not require it.

Adapt the structure to the evidence and the incident.

---

# Confidence and Uncertainty

## 45. Confidence must track evidence

When making causal claims, explicitly consider confidence.

Use:

HIGH CONFIDENCE

when supported by strong direct evidence.

MEDIUM CONFIDENCE

when supported by multiple observations but with meaningful uncertainty.

LOW CONFIDENCE

when the explanation is plausible but evidence is incomplete.

UNKNOWN

when available evidence cannot establish the claim.

Do not increase confidence simply because an explanation is elegant or familiar.

---

## 46. Evidence needed to resolve UNKNOWN

When marking something UNKNOWN, whenever useful identify:

- what evidence is missing
- where that evidence might exist
- whether it is still recoverable
- what experiment could distinguish hypotheses
- whether the uncertainty materially affects corrective actions

Do not invent certainty merely because the investigation is inconvenient.

---

# Final Principles

When analyzing an incident, remember:

1. The goal is not to find someone to blame.
2. Blameless does not mean causeless.
3. Facts come before theories.
4. Timeline comes before explanation.
5. UNKNOWN is a valid state.
6. A trigger is not a complete system explanation.
7. Local correctness does not imply system safety.
8. Automation changes where complexity lives.
9. Humans are part of the control system.
10. Work-as-done matters as much as work-as-imagined.
11. Recovery is a first-class system property.
12. Exit 0 does not prove desired state.
13. Rollback does not automatically mean recovery.
14. Recovery must be demonstrated with observable evidence.
15. AI agents are part of the system and the threat model.
16. Probabilistic reasoning must not automatically become unlimited authority.
17. Deterministic boundaries provide local guarantees, not global safety.
18. Controls must be analyzed for their interactions.
19. Corrective actions must be verifiable.
20. Important learning should become executable evidence.
21. Incidents must improve the future system.
22. Human operational knowledge remains valuable.
23. Automation should remove toil without disconnecting humans from production reality.
24. Agent changes require evaluation, not merely new instructions.
25. Security and reliability must be analyzed together when automation has authority.
26. A successful command is not proof of a successful state transition.
27. A successful recovery action is not proof of recovered service state.
28. A clean narrative is not proof of causal understanding.
29. The system includes people, tools, policies, incentives, and organizational constraints.
30. The objective is not to prevent every possible failure.
31. The objective is to build a system capable of detecting, understanding, containing, recovering from, and learning from failures that will inevitably occur.

---

# Meta-Rule

If a conclusion appears too simple, ask:

    "What system condition made this possible?"

If the explanation ends with a person:

    continue looking for the conditions that made the decision
    reasonable or possible.

If the explanation ends with a component:

    analyze its interactions with the rest of the system.

If the explanation ends with:

    "the code worked"

then:

    verify the actual state produced.

If the explanation ends with:

    "rollback worked"

then:

    verify actual recovery.

If the explanation ends with:

    "we added a control"

then:

    demonstrate what failure mode it prevents, detects, contains,
    or recovers.

If the explanation ends with:

    "the AI made a bad decision"

then:

    reconstruct context, tools, permissions, evidence,
    policies, guardrails, feedback, and human interaction
    before attributing causality to the model.

If the explanation ends with:

    "the operator made a mistake"

then:

    reconstruct what the operator knew, what they did not know,
    what alternatives existed, what constraints existed,
    and why the action made sense at the time.

If the explanation ends with a corrective action:

    ask how its implementation will be verified,
    how its risk reduction will be demonstrated,
    and what new failure modes it introduces.

If evidence conflicts:

    preserve the conflict rather than silently resolving it.

If evidence is insufficient:

    mark the claim UNKNOWN rather than inventing certainty.

The transversal question is:

    "After this incident, are the system and the people operating it
     genuinely in a better position to face the next failure,
     even if that failure occurs in a way we have not yet imagined?"