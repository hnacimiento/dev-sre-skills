# sre-incident-review — Reference: Causal, Control-System, and Human-Factors Analysis (sections 8-17)

This file is loaded on demand from `sre-incident-review/SKILL.md`'s
load-on-demand index, before naming a root cause or a human-error contributor. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-incident-review's own core has already been read.

---

# Causal Analysis

## 8. Avoid a single root cause

Do not assume that every incident has one root cause.

Analyze multiple layers:

- trigger
- immediate mechanism
- contributing conditions
- failed controls
- missing controls
- successful controls
- latent conditions
- human factors
- dependencies
- coupling
- feedback loops
- design decisions
- organizational constraints
- operational assumptions
- recovery conditions

A strong explanation may contain multiple causes and interacting conditions.

---

## 9. Distinguish causal levels

Explicitly distinguish:

TRIGGER

- the event that initiated the deviation

IMMEDIATE MECHANISM

- the mechanism that produced the observed impact

CONTRIBUTING CONDITIONS

- conditions that enabled or amplified the mechanism

FAILED CONTROLS

- controls that should have prevented, detected, contained, or stopped the failure

LATENT CONDITIONS

- structural conditions that existed before the incident

RECOVERY CONDITIONS

- conditions that enabled mitigation or recovery

Do not stop at:

    "A deployment introduced a bug."

Continue asking:

    Why was it allowed into production?

    What validation existed?

    What did the validation actually guarantee?

    What did it not guarantee?

    Why did the impact reach users?

    What signals existed?

    Why were those signals insufficient?

    What allowed the impact to expand?

    What limited the blast radius?

    What enabled recovery?

---

## 10. Trigger is not system explanation

A trigger describes where the deviation started.

It does not necessarily explain:

- why the system was vulnerable
- why the failure propagated
- why detection was delayed
- why mitigation was difficult
- why recovery succeeded or failed
- why existing controls were insufficient

Always distinguish the initiating event from the conditions that made the incident possible.

---

# Control-System Thinking

## 11. Analyze control loops

When automation, remediation, orchestration, policy engines, or AI agents are involved, analyze the incident as a control system.

Identify:

- controller
- controlled system
- observed signal
- control action
- feedback
- feedback delay
- activation conditions
- inhibition conditions
- thresholds
- limits
- state transitions
- competing controllers
- human intervention
- interaction between controllers

Ask:

    What was the controller trying to accomplish?

    What state did it believe the system was in?

    What state was the system actually in?

    What feedback did it receive?

    Was that feedback delayed, incomplete, misleading, or ambiguous?

Do not assume that a locally correct controller produces globally safe behavior.

Name a specific, recurring shape of this failure explicitly: a positive feedback loop, where a controller's corrective action makes the condition it is reacting to worse rather than better, and therefore triggers more of the same corrective action. The canonical instance is an autoscaler reacting to CPU by adding instances, where the new instances increase load on a shared, saturating dependency (a database, a downstream service) rather than relieving it — CPU stays high or climbs, so the autoscaler adds still more instances, accelerating the saturation it is indirectly causing. Every individual scaling decision can be correct according to the autoscaler's own contract (CPU was above threshold, so it added capacity) while the loop as a whole drives the system further from recovery. When reviewing a control loop, explicitly check whether the controller's own action is coupled, through some shared resource, to the signal it is observing — that coupling is what turns a normal negative-feedback control loop into a runaway positive one.

---

## 12. Local correctness is not system safety

A component may:

- satisfy its specification
- pass its tests
- return exit 0
- satisfy local invariants
- perform exactly the action it was designed to perform

and still contribute to a system-level loss of control.

Explicitly search for:

- unexpected interactions
- races
- positive feedback loops
- delayed feedback
- concurrent actions
- conflicting controllers
- state transitions
- partial states
- context changes
- locally correct but globally dangerous actions
- emergent behavior

When appropriate, apply STPA-style thinking:

    "What control actions could become unsafe under specific
     system conditions even if the controller behaves according
     to its local design?"

Do not limit the analysis to component failures.

---

# Human Factors

## 13. Work-as-Imagined vs Work-as-Done

Compare:

WORK-AS-IMAGINED

- how designers expected the work to be performed
- how documentation describes the process
- what the nominal workflow assumes

WORK-AS-DONE

- how the work was actually performed
- what adaptations were made
- what workarounds were used
- what constraints were encountered

Look for differences in:

- procedures
- tools
- permissions
- timing
- communication
- decision-making
- shortcuts
- workarounds
- coordination
- escalation

Do not automatically classify deviation from procedure as a violation.

Ask:

    "What property of the environment made this adaptation reasonable?"

---

## 14. Human decision context

For critical decisions reconstruct:

- what the operator knew
- what the operator did not know
- what signals were visible
- what signals were absent
- what alternatives existed
- what time pressure existed
- which actions were reversible
- which actions were irreversible
- what consequences were expected
- what feedback was received

The objective is to understand decisions without judging them using information that became available only later.

---

## 15. Automation bias

Analyze whether humans:

- trusted automation excessively
- assumed a tool had verified something it did not verify
- interpreted exit 0 as real success
- accepted an automated recommendation without sufficient evidence
- ignored contradictory signals
- assumed an automated rollback implied recovery

Also analyze the inverse:

- operator distrust of correct automation
- unnecessary manual intervention
- manual actions conflicting with automated controllers
- disabling automation without understanding its control role
- repeated manual intervention causing feedback conflicts

Both forms matter.

---

# Automation and Complexity

## 16. Automation changes where complexity lives

For every important automation, ask:

    "What complexity did this automation remove?"

Then ask:

    "What complexity did it move somewhere else?"

Analyze whether automation:

- simplified execution
- hid complexity
- moved complexity into diagnosis
- increased recovery difficulty
- introduced dependencies
- created new failure modes
- changed the operator's mental model
- reduced or increased cognitive load

Do not assume:

    more automation = more reliability

Automation should be evaluated by its effect on the complete sociotechnical system.

---

## 17. Clumsy automation

Identify whether automation:

- reduces work during normal conditions
- increases cognitive load during incidents
- produces surprising actions
- obscures system state
- executes multiple actions simultaneously
- is difficult to stop
- produces excessive but low-value output
- hides important decisions
- makes recovery harder
- creates coordination problems

Evaluate automation under stressed operating conditions, not only under happy-path conditions.

A specific and easy-to-miss sub-case deserves its own check: automation whose remediation action destroys the evidence needed to diagnose the problem it is remediating. Restarting instances can wipe ephemeral local logs before an operator has read them; rotating a container can discard the exact state that would have explained the failure; an automated cleanup step can delete the temporary files a human investigator needed. This is distinct from generically "increasing cognitive load" — it is the automation actively consuming the operator's diagnostic evidence as a side effect of trying to help. When reviewing an automated remediation, ask explicitly whether its own action removes evidence a human investigating the same incident would need, and if so, whether that evidence can be captured before the remediation runs rather than after.

---

