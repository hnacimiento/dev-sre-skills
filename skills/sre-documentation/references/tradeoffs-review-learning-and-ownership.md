# sre-documentation — Reference: Regression Surface, Tradeoffs, Review, Learning Loops, and Ownership (sections 38-54)

This file is loaded on demand from `sre-documentation/SKILL.md`'s
load-on-demand index, before publishing, reviewing, or considering documentation finished, or when deciding what an agent may learn from it. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-documentation's own core has already been read.

---

# Documentation as a Regression Surface

## 38. Connect Important Claims to Tests

Where practical connect documentation to executable evidence.

Examples:

    "safe to run twice"
        → idempotency test

    "rollback restores previous state"
        → recovery test

    "does not modify database"
        → scope verification

    "secret absent from artifacts"
        → security regression test

    "deployment is verified"
        → post-deployment validation

    "agent cannot perform destructive action"
        → authorization test

Documentation should remain aligned with executable reality.

---

# Documentation of Tradeoffs

## 39. Explain Why

Do not document only what the system does.

Document important reasons.

Examples:

    Why is the lock scoped this way?

    Why is the backup generated before mutation?

    Why is verification performed twice?

    Why is a degraded result acceptable?

    Why is a particular dependency required?

    Why is a manual approval required?

    Why is a limitation intentionally accepted?

This protects safety properties from future "simplification."

---

## 40. Avoid Cargo-Cult Warnings

Avoid generic warnings such as:

    WARNING: BE CAREFUL.

Instead explain:

- actual risk
- affected resource
- consequence
- mitigation

Prefer:

    CustomCss is replaced rather than merged.
    A backup is created before mutation and can be restored
    using the generated rollback artifact.

Specific warnings are operationally useful.

---

# Public Repository Quality

## 41. Public Repositories Should Optimize For Trust

For public GitHub repositories prioritize:

- clarity
- reproducibility
- safe defaults
- transparent limitations
- understandable architecture
- easy onboarding
- explicit prerequisites
- recovery guidance
- security transparency
- current examples

Do not assume readers know:

- the author's environment
- internal infrastructure
- undocumented conventions
- hidden prerequisites
- organizational assumptions

---

# Documentation Review Procedure

## 42. Review Against Reality

When reviewing documentation compare it against:

- implementation
- tests
- configuration
- generated artifacts
- deployment behavior
- runtime behavior
- operational tooling
- security controls

Look specifically for:

- promises without implementation
- implementation without documentation
- stale examples
- missing failure paths
- missing recovery
- broad security claims
- incorrect exit-code semantics
- undocumented side effects
- undocumented generated artifacts
- outdated permissions
- unsupported guarantees

---

## 43. Review High-Risk Claims First

Prioritize review of claims involving:

- destructive operations
- credentials
- security
- data integrity
- availability
- recovery
- rollback
- automation
- idempotency
- concurrency
- agent authority
- blast radius

The higher the consequence of a false claim, the stronger the evidence should be.

---

# Documentation Completion Gate

## 44. Publication Checklist

Before publishing important operational documentation verify:

    [ ] Scope is explicit
    [ ] Audience is clear
    [ ] Prerequisites are explicit
    [ ] Resources affected are documented
    [ ] Destructive behavior is visible
    [ ] Failure states are documented
    [ ] Recovery is documented
    [ ] Verification scope is explicit
    [ ] Exit codes are accurate
    [ ] Generated artifacts are documented
    [ ] Security claims are precise
    [ ] Credentials and secrets are addressed
    [ ] Limitations are visible
    [ ] Architecture matches implementation
    [ ] Examples are current
    [ ] Important claims have evidence
    [ ] Idempotency claims are justified
    [ ] Blast radius is documented where relevant
    [ ] Agent permissions are documented where relevant
    [ ] Control boundaries are documented where relevant
    [ ] Documentation does not promise more than the implementation proves

---

# Documentation Quality Gate

## 45. The Four Operational Questions

High-quality documentation should allow a person who did not write the system to answer:

    What will this change?

    What can go wrong?

    How will I know what happened?

    How do I recover?

Also answer:

    What does the system NOT guarantee?

If these questions cannot be answered, the documentation is incomplete.

---

# Anti-Patterns

## 46. Detect Documentation Anti-Patterns

Treat the following as warning signs:

- documentation describes only the happy path
- claims are broader than evidence
- "safe" is used without defining scope
- "idempotent" is used without a contract
- rollback is documented without verification
- exit 0 is described as system success
- recovery instructions omit prerequisites
- destructive behavior is hidden
- security claims are absolute without evidence
- generated artifacts are undocumented
- limitations are omitted
- examples are stale
- architecture does not match implementation
- runbooks require guessing
- warnings are generic
- operational procedures contain commands without expected results
- documentation assumes undocumented internal knowledge
- AI recommendations are presented as authoritative
- agent permissions are undocumented
- human approval is treated as an unconditional safety guarantee
- deterministic controls are presented as proof of global safety
- postmortem learning remains only in prose
- documentation is updated without validating implementation
- implementation changes occur without reviewing documentation impact

---

# Change Impact

## 47. Documentation Must Participate in Change Design

When changing a system ask:

    What documentation becomes false if this change is merged?

Consider:

- README
- architecture
- runbooks
- recovery
- security
- release documentation
- incident procedures
- generated examples
- agent behavior
- operational expectations

Documentation impact is part of engineering impact.

---

# Learning Loop

## 48. Documentation Should Feed the Operational System

Documentation should not be the final destination of learning.

Important operational learning should become, where appropriate:

- code
- tests
- invariants
- policies
- alerts
- validation
- runbooks
- simulations
- fault injection
- GameDays
- recovery drills
- agent evaluations
- architecture changes

Conceptually:

    INCIDENT
        ↓
    EVIDENCE
        ↓
    HUMAN ANALYSIS
        ↓
    DOCUMENTED LEARNING
        ↓
    SYSTEM CHANGE
        ↓
    EXECUTABLE VALIDATION
        ↓
    FUTURE OPERATIONS
        ↓
    NEW EVIDENCE

Documentation is therefore part of the feedback loop.

---

## 49. Human Operational Knowledge Matters

Important operational knowledge may exist only in human experience:

- hypotheses
- ambiguous signals
- decision context
- tradeoffs
- workarounds
- discoveries
- failed approaches
- unexpected interactions

Preserve this knowledge when it is useful.

Do not reduce operational documentation to command lists.

The purpose is knowledge transfer across time and people.

---

# AI Learning Loop

## 50. Do Not Feed Incidents to Agents Blindly

If incident documentation is used to improve an AI agent, distinguish:

- verified facts
- hypotheses
- successful actions
- failed actions
- contextual actions
- assumptions
- environmental conditions
- operator decisions

Do not convert:

    "This worked during this incident."

into:

    "This must always be done."

Incident knowledge is contextual.

---

## 51. Evaluate Agent Changes

If an incident causes a change to:

- prompts
- playbooks
- policies
- tools
- agent permissions
- agent behavior
- evaluation criteria

document:

- what changed
- why it changed
- what failure mode it targets
- what scenarios it affects
- what regressions are possible
- how it will be evaluated
- what evidence demonstrates improvement

Prefer executable evaluation and deterministic boundaries around agent behavior.

---

# Ownership

## 52. Documentation Must Preserve Ownership

Make clear:

- who operates the system
- who owns the code
- who owns automation
- who owns alerts
- who owns runbooks
- who owns recovery
- who owns security controls
- who owns agents
- who validates production changes

Automation does not eliminate ownership.

An agent does not possess organizational accountability.

---

# Meta-Rules

## 53. When Documentation Is Ambiguous

If a claim sounds too simple, ask:

    What exact behavior does this statement refer to?

If a claim says:

    "safe"

ask:

    Safe from what?

If a claim says:

    "automatic"

ask:

    Which steps are actually automatic?

If a claim says:

    "verified"

ask:

    Verified against what observable state?

If a claim says:

    "recovered"

ask:

    What evidence demonstrates recovery?

If a claim says:

    "idempotent"

ask:

    What happens on the second execution?

If a claim says:

    "secure"

ask:

    Against which threat and within which boundary?

If a claim says:

    "AI-assisted"

ask:

    What authority does the agent actually have?

If a claim says:

    "read-only"

ask:

    Read-only with respect to what? A command can leave the target system
    genuinely untouched while still writing a local cache, a temp file, or
    a log entry on the machine running it — those are different claims,
    and "read-only" alone does not say which one is being made.

If a claim says:

    "fully monitored"

ask:

    Monitored for which failure modes specifically? Golden-signal coverage
    is not the same as correctness or integrity coverage (see the
    distinction in sre-observability).

If a claim says:

    "will never experience downtime"

ask:

    What evidence could possibly support a claim about something that has
    not happened yet? Prefer a stated target and a measured track record
    over an absolute guarantee no system can actually make.

If a claim says an agent or automation:

    "cannot make dangerous changes"

ask:

    Is that prevented by a deterministic boundary outside the model, or
    by the model choosing not to? Only the former supports the claim as
    written.

---

## 54. Final Principle

The highest-quality operational documentation is not documentation that makes the system sound safest.

It is documentation that makes the system's actual behavior easiest to understand, verify, operate, recover, and improve.

A person who did not build the system should be able to determine:

    What will this change?

    What can go wrong?

    What state might remain?

    How will I know?

    How do I recover?

    What does the system NOT guarantee?

    Who owns the outcome?

The ultimate goal is operational truth.

Documentation should reduce ambiguity without hiding uncertainty, preserve safety boundaries without creating false confidence, and turn operational knowledge into reusable evidence for the next cycle of engineering.