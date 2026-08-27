# sre-engineering-mindset — Reference: Execution and Closing (Sections 31–43)

This file is loaded on demand from `sre-engineering-mindset/SKILL.md` §1c,
before treating any consequential automation as finished, once the
situation scored above the floor in §1a. It is not a standalone skill —
it assumes the calibration in the parent `SKILL.md` has already run.

---

# 31. Dry Run and Preview

Dry-run is valuable when it improves:

- safety;
- operator understanding;
- confidence;
- reviewability;
- prediction of blast radius.

But a dry-run is not automatically equivalent to the real operation.

Ask:

- Does dry-run use the same selection logic?
- Does it inspect the same state?
- Can state change between preview and execution?
- Does it expose the actual resources that will be modified?
- Are destructive consequences clearly visible?

Do not create a decorative dry-run that provides false confidence.

---

# 32. Testing the System, Not Just the Code

Tests should cover:

    expected behavior
    +
    failure behavior
    +
    recovery behavior
    +
    concurrency
    +
    repeated execution
    +
    environmental surprises
    +
    operator interaction
    +
    security abuse

Use fault injection where practical.

Ask:

    "What assumption does this test prove?"

and:

    "What important assumption remains untested?"

For agentic systems, include non-deterministic evaluation and adversarial
scenarios.

Specialized testing belongs in `sre-testing`.

---

# 33. Documentation Is an Operational Interface

Documentation is part of the system.

A README should not merely explain what the software does.

It should communicate:

- scope;
- prerequisites;
- assumptions;
- blast radius;
- mutation points;
- failure modes;
- recovery;
- limitations;
- security model;
- operational expectations;
- what the system does NOT guarantee.

Documentation must reflect actual behavior.

Never document an intended guarantee that the implementation cannot demonstrate.

When documentation and code disagree:

    treat the discrepancy as an engineering defect.

---

# 34. Releases and Deployment

A reliable change is not merely a successful build.

Consider:

- rollout strategy;
- compatibility;
- rollback;
- observability;
- progressive exposure;
- failure detection;
- blast radius;
- dependency behavior;
- migration state;
- operator readiness.

Prefer small, reversible changes.

The safer the rollback and verification mechanisms, the safer the deployment
strategy can be.

Specialized release guidance belongs in `sre-release-deployment`.

---

# 35. Incident Thinking

During incidents:

    stabilize first
    understand continuously
    communicate clearly
    preserve evidence
    avoid making the system harder to recover

Do not force a rigid checklist onto a dynamic incident.

Use structured thinking without assuming the incident itself is structured.

Update the model continuously as evidence changes.

Afterward, convert surprises into:

- tests;
- instrumentation;
- automation improvements;
- documentation;
- architectural changes;
- training;
- policy changes.

Specialized incident-review guidance belongs in `sre-incident-review`.

---

# 36. Ownership

Automation can execute.

Automation can recommend.

Automation can detect.

Automation can recover.

But organizational accountability remains human.

There must be a clear answer to:

    Who owns this system?

    Who is authorized to change it?

    Who is responsible for its reliability?

    Who responds when it fails?

    Who decides when automation must be disabled?

Avoid creating "orphan automation" that everyone depends on but nobody owns.

---

# 37. Do Not Overfit to Google

Google's SRE practices provide powerful principles and examples.

Do not cargo-cult infrastructure that only makes sense at Google's scale.

Adapt mechanisms to:

- system criticality;
- organizational maturity;
- team size;
- threat model;
- regulatory environment;
- operational complexity;
- available tooling.

However, adaptation must not become an excuse to discard the underlying
reliability principle.

Distinguish:

    principle

from:

    implementation mechanism.

For example:

    Principle:
        high-risk destructive operations require strong safeguards.

    Mechanism:
        a specific Google-style authorization system.

The first may be universal.

The second may not be.

---

# 38. Avoid Absolute Rules Without Context

Be suspicious of statements such as:

    "always"

    "never"

    "must"

unless the statement represents a genuine safety invariant.

A good SRE framework distinguishes:

### Hard invariant

Violation is unacceptable.

### Strong default

Normally preferred unless a documented reason exists.

### Heuristic

Useful guidance, not a universal law.

### Context-dependent mechanism

Chosen according to environment.

This prevents the framework from becoming dogmatic.

### State Your Confidence Explicitly, Especially Off the Shell

Not every claim in this framework carries the same weight of evidence.
Statements about shell semantics, exit codes, or file atomicity are close
to verifiable fact: you can demonstrate them by running the script. Claims
about organizational practice — how much toil is sustainable, whether a
team is ready for a given process, what a "healthy" on-call rotation looks
like, whether a cultural pattern is an anti-pattern in this specific
organization — are far more contested, context-dependent, and easier to
get wrong by projecting a general rule onto a specific team that does not
match it.

When applying this mindset, distinguish out loud between "this is a
demonstrable technical property" and "this is a generalization from
common practice that may not fit your situation," and say so plainly
rather than presenting both with the same tone of authority. This matters
most in any specialized skill that reasons about SLOs, error budgets, toil
ratios, on-call load, or organizational adoption: those numbers exist in
the literature as governance signals and rough heuristics, not as
universal targets, and a skill that repeats them should carry that caveat
forward rather than launder them into hard requirements. Treat this as a
standing instruction: when in doubt about whether a claim is solid ground
or a judgment call, say which one it is and give the person enough to
decide for themselves.

---

# 39. Resolving Conflicts Between Principles

SRE principles will sometimes conflict.

Examples:

    safety vs speed

    automation vs human understanding

    least privilege vs incident flexibility

    deterministic control vs operational adaptability

    simplicity vs resilience

    standardization vs local optimization

    rollback safety vs recovery speed

    automation vs operator learning

When principles conflict:

1. Identify the actual system property at risk.
2. Identify the failure mode each principle is protecting against.
3. Determine whether the conflict is real or caused by implementation choices.
4. Prefer solutions that preserve both properties where possible.
5. If a tradeoff is unavoidable, make it explicit.
6. Bound the downside.
7. Add observability.
8. Define how the decision can be revisited.
9. Document the rationale when the tradeoff is operationally significant.

Never resolve conflicts merely by selecting the most convenient principle.

---

# 40. The SRE Decision Loop

Do not treat this as a rigid sequence.

Use it as a continuously revisited reasoning loop:

    OBSERVE
       ↓
    UNDERSTAND
       ↓
    DEFINE INTENT
       ↓
    IDENTIFY RISKS
       ↓
    DESIGN CONTROL
       ↓
    MUTATE
       ↓
    VERIFY
       ↓
    OBSERVE AGAIN
       ↓
    LEARN
       ↺

At any point, new evidence can invalidate the previous model.

Therefore:

    the model is provisional.

The system is allowed to surprise you.

Your design must make surprises survivable.

---

# 41. The Final SRE Questions

Before considering consequential automation complete, ask:

### Problem

- What problem are we actually solving?
- Is this reducing toil or merely moving it?

### State

- What state exists now?
- What state do we want?
- How do we know?

### Risk

- What is the blast radius?
- What is irreversible?
- What happens if selection is wrong?

### Failure

- What happens if it fails halfway?
- What happens if the environment changes?
- What happens if the system behaves correctly but the overall result is unsafe?

### Recovery

- Can we recover?
- Is recovery itself fallible?
- Can recovery prove that it succeeded?

### Verification

- Are we verifying postconditions or merely command exit codes?

### Humans

- What will operators stop learning?
- Can they still understand and recover the system?

### Automation

- Does automation reduce toil without creating clumsy automation?
- Can operators inspect and predict its actions?

### Agents

- If an AI agent is involved, what can it observe?
- What can it change?
- Who authorizes it?
- What happens if its reasoning is wrong?
- Can untrusted input manipulate it?
- How is its behavior evaluated?
- How is drift detected?

### Security

- Can an attacker trigger the same failure?
- Can credentials or authority escape the intended boundary?

### System

- Are components correct but their interactions unsafe?
- What control loop exists?
- Where is the feedback?

### Learning

- What will we learn from this operation?
- How will that learning improve the system?

---

# 42. What This Skill Does NOT Do

This skill establishes the reasoning framework.

It does not attempt to contain every implementation detail.

Use specialized skills for:

    sre-bash
        Shell-specific reliability, Bash semantics, traps, quoting,
        subprocess behavior, portability, shellcheck, etc.

    sre-testing
        Test strategy, fault injection, regression testing, test matrices,
        deterministic and non-deterministic evaluation.

    sre-security
        Threat modeling, secrets, authorization, least privilege,
        attack surfaces, prompt injection, security controls.

    sre-observability
        Metrics, logs, traces, events, telemetry, alerting, SLOs.

    sre-release-deployment
        Rollouts, progressive delivery, migrations, rollback,
        release safety.

    sre-documentation
        Operational documentation, contracts, runbooks, architecture docs.

    sre-incident-review
        Incident response, postmortems, learning, timelines,
        organizational improvement.

    sre-slo
        Service level objectives, error budgets, burn-rate alerting,
        freeze policy.

The mindset should coordinate these skills rather than duplicate them.

---

# 43. Definition of Done for the Mindset

A solution should not be considered SRE-mature merely because:

- the code works;
- tests pass;
- deployment succeeds;
- documentation exists;
- automation is present;
- an agent produces the correct recommendation.

A mature solution demonstrates that:

1. The intended state is explicit.
2. Mutations are bounded.
3. Failure states are understood.
4. Recovery semantics are explicit.
5. Success is verified.
6. Partial failure is visible.
7. Humans can understand the system.
8. Security boundaries are deliberate.
9. Automation does not silently create new operational toil.
10. Agentic behavior, when present, is bounded and evaluated.
11. System-level interactions have been considered.
12. Production feedback can improve the system.
13. Documentation matches actual behavior.
14. Ownership is clear.
15. Important assumptions are either enforced or tested.

The standard is not perfection.

The standard is:

    "When reality disagrees with our assumptions,
     does the system fail safely, tell us the truth,
     and leave us in a position from which we can recover?"

That is the core SRE mindset.
