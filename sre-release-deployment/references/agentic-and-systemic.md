# sre-release-deployment — Reference: Agentic Release Operations and Systemic Risk (sections 37-52)

This file is loaded on demand from `sre-release-deployment/SKILL.md` §1a, whenever an AI agent triggers or assists a release, or the blast radius is high enough that component correctness cannot be trusted to imply system-level safety. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-release-deployment's own core has
already been read.

---

# 37. Human Factors

A deployment interface is an operational interface.

At 03:00, the operator needs to understand:

- what is happening
- what changed
- what failed
- what is safe to do next
- whether the system is still mutating
- whether rollback is running
- whether the result is known

Do not optimize only for developer convenience.

Optimize for recoverability under stress.

---

# 38. Work-as-Done

Do not design releases solely around how you imagine operators will use them.

Observe:

- how people actually deploy
- how they bypass controls
- where they copy/paste
- where they improvise
- what commands they run during incidents
- what information they search for
- which safeguards they disable under pressure

If operators routinely bypass a mechanism, treat that as design feedback.

Do not automatically classify the operator as the problem.

---

# 39. Automation Must Reduce Toil Without Removing Operational Awareness

Automation should remove repetitive mechanical work.

It should not make the responsible engineers completely disconnected from
production reality.

Preserve:

- meaningful visibility
- operational learning
- incident participation
- ability to inspect state
- ability to understand automated decisions
- ability to recover manually when necessary

Automation should increase human capacity, not eliminate human situational
awareness.

---

# 40. Agentic Release Operations

When an AI agent participates in deployment, treat it as another component
of the production control system.

The agent is not automatically authoritative.

Separate:

    PROPOSED
    APPROVED
    EXECUTED
    VERIFIED

Do not collapse these into:

    AGENT_SUCCESS

The agent may:

- inspect
- correlate
- summarize
- propose
- generate plans
- recommend rollback
- invoke constrained tools

But high-risk mutation should be governed by deterministic policy and
appropriate authorization.

---

# 41. Agent Outputs Are Not Evidence

Never use:

    "The agent says deployment succeeded."

as sufficient verification.

Agent output is an observation/proposal.

Independent deterministic mechanisms should verify critical state.

For example:

    agent proposes deployment
        ↓
    policy validates action
        ↓
    authorized tool executes
        ↓
    deterministic verification checks state
        ↓
    system records VERIFIED

---

# 42. Deterministic Guarantees

Use probabilistic systems where they provide useful adaptation.

Use deterministic mechanisms where guarantees matter.

Prefer deterministic mechanisms for:

- hashing
- integrity checks
- authorization
- schema validation
- exact comparisons
- resource selection constraints
- safety limits
- state transitions
- invariant enforcement
- destructive-operation interlocks

Do not use an LLM where a small deterministic validator can provide a stronger
guarantee.

---

# 43. Agent Threat Model

An agent with production access is part of the security threat model.

Consider:

- prompt injection
- malicious logs
- malicious tickets
- malicious configuration
- poisoned documentation
- untrusted external content
- compromised tools
- manipulated telemetry
- credential exposure
- excessive permissions
- tool abuse
- confused deputy behavior
- authorization bypass
- accidental broad targeting
- data exfiltration
- context poisoning

Never allow external text to silently become trusted control instructions.

Keep tool permissions narrowly scoped.

(Maintenance note: sre-engineering-mindset §19 and sre-security's Agent
Threat Model / Tool Boundaries sections carry adapted versions of this
same threat list. When adding a new threat category here, add it there
too rather than letting the lists silently diverge.)

---

# 44. Agent Tool Boundaries

A safe architecture should distinguish:

    reasoning

from:

    authorization

from:

    execution

from:

    verification

An agent should not be able to transform arbitrary natural-language reasoning
directly into unrestricted production mutation.

Prefer:

    agent
      ↓
    constrained typed tool
      ↓
    policy/interlock
      ↓
    authorization
      ↓
    execution
      ↓
    deterministic verification

---

# 45. STPA / System-Level Thinking

Do not assume:

    component A is correct
    component B is correct
    therefore system is safe

Consider interactions.

Ask:

- Can two individually valid controllers conflict?
- Can automation race with a human?
- Can rollback race with rollout?
- Can health checks lag behind mutation?
- Can stale telemetry trigger a valid but unsafe action?
- Can a controller interpret temporary absence as permanent failure?
- Can retries amplify a partial failure?
- Can safety checks themselves become stale?

Reliability is a property of the control system, not merely its components.

---

# 46. Safety Invariants

Define properties that must remain true regardless of deployment strategy.

Examples:

- never mutate resources outside target scope
- never expose credentials
- never exceed maximum blast radius
- never deploy an unverified artifact
- never claim success without required verification
- never perform destructive action against an ambiguous target
- never allow rollback to silently report success after partial recovery
- never let an agent bypass authorization
- never treat unknown state as confirmed success

These are system-level safety properties.

---

# 47. Failure Injection

Release mechanisms should be tested against realistic failure.

Test:

- artifact unavailable
- artifact corrupted
- dependency unavailable
- authentication failure
- authorization failure
- target disappears
- network timeout
- process termination
- partial rollout
- partial rollback
- verification failure
- stale telemetry
- concurrent deployment
- duplicate invocation
- SIGINT/SIGTERM
- SIGKILL
- storage failure
- configuration mismatch
- schema incompatibility
- agent recommendation error
- prompt injection
- policy rejection

The objective is not merely:

    did the script fail?

It is:

> Did the system fail safely, observably, and recoverably?

---

# 48. Kill -9 and Process Death

Do not design only for graceful failure.

A deployment process can disappear because of:

- SIGKILL
- OOM
- host failure
- power loss
- terminal loss
- container destruction
- network partition

Ask:

> If the process disappears at every mutation point, what state remains?

Recovery artifacts should exist before they are needed.

Important state should be durable.

The system should support reconciliation after process loss.

---

# 49. Generated Recovery Artifacts

If deployment generates:

- rollback scripts
- manifests
- state files
- recovery metadata
- migration plans

treat them as production artifacts.

Validate them before the mutation begins when possible.

Record:

- generation source
- version
- timestamp
- target
- dependencies
- integrity
- expected usage

Generated recovery code must be independently executable and independently
observable.

Do not assume the generating process will still exist when recovery is needed.

---

# 50. Release State Must Survive Process Loss

Do not keep critical recovery information only in process memory.

Persist what is needed to determine:

- whether mutation started
- what artifact was selected
- what targets were affected
- what completed
- what remains
- what recovery artifact applies
- what final state was verified

A process dying must not erase the operational truth.

---

# 51. Rollback Artifacts Must Be Tested

Do not assume generated rollback logic works because the template compiled.

Test the generated artifact itself.

At minimum:

- syntax validation
- static analysis
- happy path
- partial failure
- missing dependency
- missing credentials
- target disappearance
- corrupted restore
- verification failure
- interrupted execution

The artifact that runs during the incident is the artifact that matters.

---

# 52. Release Safety and Supply Chain

Treat the release path itself as part of the trusted computing base.

Protect against:

- compromised dependencies
- artifact substitution
- mutable downloads
- unsigned artifacts
- compromised build systems
- malicious CI changes
- dependency confusion
- unauthorized release
- compromised deployment credentials

The deployment mechanism must establish what it is executing.

---

