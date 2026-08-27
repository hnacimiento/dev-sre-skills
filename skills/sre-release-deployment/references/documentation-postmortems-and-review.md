# sre-release-deployment — Reference: Documentation, Postmortems, and Review (sections 53-64)

This file is loaded on demand from `sre-release-deployment/SKILL.md` §1a, before considering a release process, runbook, or review finished. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-release-deployment's own core has
already been read.

---

# 53. Documentation as Operational Interface

Release documentation should allow an engineer unfamiliar with the system to
answer quickly:

- what does this release change?
- what can it break?
- how do I verify it?
- what does failure look like?
- how do I stop it?
- how do I recover?
- what is irreversible?
- what evidence should I collect?

Documentation is part of reliability.

---

# 54. Release Runbooks

A good runbook should not merely say:

    run command X

It should explain:

    Preconditions
    Expected observation
    Action
    Verification
    Failure interpretation
    Recovery
    Escalation

Avoid runbooks that force operators to infer state from ambiguous output.

---

# 55. Incident Interaction

During an incident, deployment state becomes part of the incident timeline.

Preserve:

- deployment start
- rollout progression
- observed symptoms
- operator decisions
- automated decisions
- aborts
- rollback
- verification
- final state

Deployment evidence should correlate with incident evidence.

---

# 56. Postmortem Feedback Loop

Every significant release failure should improve the release system.

Ask:

- Why was the failure possible?
- Why was it not detected earlier?
- Why did verification fail or not exist?
- Why was rollback insufficient?
- Why was the blast radius what it was?
- Why did the operator choose that action?
- Did the automation help or hinder?
- Did documentation match reality?
- Which guardrail should become deterministic?
- Which test should be added?
- Which recurring manual step is toil?

The objective is not merely to document the failure.

The objective is to improve the system that produced the failure.

---

# 57. Postmortem Nutrition Loop

Operational incidents produce valuable data.

That data should feed:

    incident
      ↓
    timeline
      ↓
    postmortem
      ↓
    failure modes
      ↓
    tests
      ↓
    safeguards
      ↓
    automation
      ↓
    improved release system

If AI agents participate in operations, the loop may also become:

    production
      ↓
    human judgment
      ↓
    postmortem
      ↓
    curated operational knowledge
      ↓
    agent evaluation / playbook improvement
      ↓
    safer automation

Do not allow automation to eliminate the human feedback required to improve
the automation itself.

This loop has a specific failure mode worth naming: as release automation
absorbs more of the response to a failed deploy, operators get less
first-hand practice diagnosing what actually went wrong, and the
postmortems that feed this loop get thinner as a result — fewer discarded
hypotheses, less hard-won context. Thinner postmortems are exactly the
input this loop and any agent-evaluation loop built on it depend on. This
is the same mechanism documented from the observability angle in
sre-observability §58 and from the incident-review angle in
sre-incident-review's Postmortem Nutrition Loop section — treat it as one
phenomenon described three times, from three angles, not three separate
risks. (Maintenance note: this skill, sre-observability, and
sre-incident-review each keep their own version of this loop, adapted to
their own domain — when revising the underlying concept, check the other
two rather than assuming this copy is the only one.)

---

# 58. Release Metrics Must Not Become Targets

Metrics can be gamed unintentionally.

Examples:

- maximizing deployment frequency
- minimizing rollback count
- maximizing deployment success percentage
- minimizing change duration

without considering:

- correctness
- incidents
- escaped failures
- hidden partial failures
- customer impact

Measure outcomes, not activity alone.

---

# 59. Anti-Patterns

Reject or question designs that:

- equate exit 0 with release success
- equate deployment command success with system health
- deploy everything simultaneously without justification
- have no post-deployment verification
- have no recovery strategy
- claim rollback is guaranteed without testing it
- silently ignore partial deployment
- silently convert unknown into success
- retry unsafe operations blindly
- rely on mutable artifact identity
- expose secrets in release logs
- depend entirely on process memory
- trust cached target identity blindly
- allow uncontrolled concurrent deployment
- use LLMs for deterministic guarantees
- allow agents unrestricted production access
- treat agent output as verification
- assume healthy components imply a healthy system
- make destructive changes equivalent to constructive changes
- make rollback a hidden side effect
- rely on undocumented operator knowledge
- optimize deployment velocity at the expense of reliability

## 59a. The Checklist Trap

A specific, recurring argument deserves its own reference: "we have
canary, rollback, healthchecks, exit codes, locks, an AI agent with
human-in-the-loop, logs, and checksums, therefore the deployment is
reliable." Every item in that list is a real, useful control. None of
them, individually or added together, proves the conclusion, because each
guarantees a narrow local property while the conclusion is a claim about
the whole system's behavior over time.

| Property | What it actually guarantees | What it does NOT guarantee |
|---|---|---|
| Canary | The sampled population showed acceptable signals for the observation window used | The signals chosen were the right ones, the population was representative, or that expansion is safe if the environment differs from the canary window |
| Rollback exists | A mechanism to attempt restoring prior state is present | That the mechanism has been exercised under real failure conditions, or that restoring old code is compatible with data changes already made |
| Healthchecks | The specific endpoint returned the specific expected response at the specific moment checked | That the service is behaving correctly for real users, or that data returned is correct (sections 11-12) |
| Exit codes | The process's own accounting of its result, as it defines that accounting | That the accounting is truthful, that it was based on verified postconditions, or that partial failure was not silently rounded up to success |
| Locks | Two holders of the same lock cannot proceed at the same time | That everything relevant is covered by the lock, or that state outside its scope cannot still race |
| Agent + human-in-the-loop | A human had an opportunity to review a proposed action | That the human received accurate, current, non-stale evidence to review, or that the target had not changed between proposal and execution (sections 41, 44) |
| Logs retained | A record exists somewhere | That the record is complete, that it does not itself leak secrets, or that anyone will look at it before the next incident |
| Checksums / SHA256 | The artifact matches a specific stored hash value | Where that hash came from, whether it is reachable by the same compromise it is meant to detect, or that the artifact's origin is authentic (see sre-security, integrity vs authenticity) |

The smallest counterexample that defeats the whole list looks like this:
every one of these controls fires exactly as designed, and the system
still loses control, because the failure lives in an interaction the
list does not cover — for example, the canary passed, rollback executed
and returned 0, the healthcheck returned 200, the lock correctly
serialized two deployments, the agent's action was approved by a human
who was shown accurate information at approval time — but the approved
target silently changed between approval and execution (section 41's
stale-approval problem), so the human approved one thing and a different
thing happened. Every control did exactly its job. The system still
failed, because none of those controls was responsible for the specific
interaction that broke.

The operative principle from this skill that prevents the false sense of
security is section 45 (STPA / System-Level Thinking): reliability is a
property of the system's interactions, not the sum of its individually
correct components. A checklist of controls tells you which local
guarantees exist. It does not, by itself, tell you whether they compose
into the global guarantee being claimed.

---

# 60. Decision Framework

When reviewing a release design, ask:

## Intent

    What are we trying to change?

## Scope

    What exactly can change?

## Preconditions

    What must be true before mutation?

## Artifact

    Exactly what are we deploying?

## Identity

    How do we know the target is correct?

## Blast radius

    What is the maximum possible impact?

## Intermediate states

    Are partial states safe?

## Mutation

    What actually changes?

## Verification

    How do we prove the intended state exists?

## Failure

    What happens at every mutation point?

## Unknown

    What happens if we lose knowledge of the state?

## Recovery

    Can we restore or reconcile safely?

## Idempotency

    What happens if the operation runs twice?

## Concurrency

    What happens if another actor changes the same system?

## Observability

    Can an operator reconstruct what happened?

## Security

    Can credentials or authorization boundaries be bypassed?

## Human factors

    Can an operator understand and recover under stress?

## Agent

    If AI participates, what can it propose, authorize, execute, and verify?

## System interactions

    Can individually correct components interact into an unsafe state?

## Learning

    How does this release process improve after failure?

---

# 61. Required Reasoning Pattern

When reviewing a release, do not immediately write code.

First establish:

    1. Desired state
    2. Current state
    3. Preconditions
    4. Target identity
    5. Mutation points
    6. Intermediate states
    7. Failure modes
    8. Verification
    9. Recovery
    10. Observability
    11. Security boundaries
    12. Human interaction
    13. Agent interaction, if applicable
    14. System-level interactions
    15. Tests
    16. Operational documentation

Then decide whether implementation is appropriate.

---

# 62. Review Severity

Use severity based on operational consequence.

P0:
    Immediate catastrophic or uncontrolled production impact.

P1:
    Serious reliability, security, recovery, or release-control failure
    capable of producing significant production impact or misleading
    operators about critical state.

P2:
    Important reliability or operational weakness with meaningful but
    contained impact.

P3:
    Lower-risk maintainability, observability, documentation, or
    defense-in-depth issue.

Severity should reflect:

    blast radius
    likelihood
    detectability
    recoverability
    duration
    reversibility

Do not assign severity based only on code complexity.

---

# 63. Review Style

When reviewing another engineer's release system:

1. Validate claims against actual behavior.
2. Distinguish documentation from implementation.
3. Identify assumptions.
4. Identify implicit state.
5. Find mutation points.
6. Find unverified transitions.
7. Find ambiguous outcomes.
8. Find recovery gaps.
9. Find dangerous retries.
10. Find blast-radius expansion paths.
11. Find concurrency races.
12. Find security boundary violations.
13. Find human-factor failure modes.
14. Find agent trust boundaries.
15. Challenge apparently safe component-level guarantees.
16. Prioritize findings by operational consequence.

Do not praise a design merely because it follows familiar DevOps patterns.

Ask whether the system can fail safely.

---

# 64. Final Principle

The goal of release engineering is not:

    deploy successfully

The goal is:

    introduce change
    without losing control of the system
    while preserving the ability to observe,
    verify,
    stop,
    recover,
    explain,
    and learn.

A strong release system does not assume that changes will always succeed.

It is designed around the expectation that changes will sometimes fail.

The mark of reliability is therefore not the absence of failure.

It is:

    controlled change
    +
    bounded blast radius
    +
    observable state
    +
    explicit uncertainty
    +
    verified outcomes
    +
    recoverability
    +
    human understanding
    +
    continuous learning

When an AI agent participates, add:

    constrained authority
    +
    deterministic guarantees
    +
    independent verification
    +
    explicit accountability
    +
    adversarial threat modeling

The release system should remain safe even when:

    the deployment fails
    the network fails
    the process dies
    telemetry becomes stale
    rollback fails
    operators improvise
    automation behaves unexpectedly
    an agent makes a bad recommendation
    an attacker supplies malicious input
    multiple control loops interact unexpectedly

That is the standard.

Do not optimize for the appearance of successful deployment.

Optimize for maintaining control of production through change.