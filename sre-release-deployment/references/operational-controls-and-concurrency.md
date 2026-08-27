# sre-release-deployment — Reference: Operational Controls and Concurrency (sections 25-36)

This file is loaded on demand from `sre-release-deployment/SKILL.md` §1a, whenever config changes, feature flags, freezes, error budgets, secrets, concurrency, or destructive changes are involved. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-release-deployment's own core has
already been read.

---

# 25. Configuration Changes

Configuration is production code.

Apply release discipline to:

- environment variables
- feature flags
- API endpoints
- routing
- timeouts
- resource limits
- authentication configuration
- security policies
- database configuration

A one-line configuration change can have a larger blast radius than a
large code deployment.

---

# 26. Feature Flags

Feature flags reduce deployment coupling but introduce operational state.

Track:

- owner
- intended lifetime
- default state
- rollout population
- dependencies
- expiration/removal plan
- observability

Avoid accumulating permanent flags.

A flag that nobody understands is operational debt.

---

# 27. Change Freeze

A freeze should be treated as a risk-control mechanism, not bureaucracy.

Use heightened restrictions when:

- system is unstable
- error budget is exhausted
- major incident is active
- observability is degraded
- dependencies are uncertain
- rollback capability is unavailable

Do not let normal deployment velocity override an already degraded
reliability state without explicit justification.

---

# 28. Error Budget

Changes consume reliability risk.

Before a significant release, consider:

- current SLO status
- recent incidents
- error-budget consumption
- deployment failure rate
- ongoing degradation
- confidence in recovery

The decision is not:

    "Can we deploy?"

It is:

    "Is this an acceptable use of the reliability budget right now?"

---

# 29. Change Failure Rate

Track release outcomes.

Useful signals include:

- deployment success rate
- change failure rate
- rollback frequency
- rollback success rate
- time to recovery
- time to detect deployment-induced degradation
- partial deployment frequency
- failed verification frequency
- escaped defects
- emergency changes
- repeated retries

Metrics should support decisions.

Do not optimize a metric in isolation.

---

# 30. Deployment Observability

Every meaningful release should produce enough evidence to answer:

    What changed?
    Who/what initiated it?
    Where?
    When?
    Why?
    Which artifact?
    Which configuration?
    Which authorization?
    What happened?
    What failed?
    What was verified?
    What remained unknown?
    What was the final state?

Use correlation identifiers to connect:

- deployment
- build
- artifact
- logs
- traces
- metrics
- configuration changes
- recovery
- incident

---

# 31. Auditability

Auditability is different from debugging.

Debug logs explain implementation behavior.

Audit records answer:

    who
    did what
    to which target
    under which authorization
    when
    with what result

Preserve operational evidence for high-risk changes.

Never place secrets into audit records.

---

# 32. Secrets

Deployment systems frequently handle secrets.

Never expose secrets through:

- command-line arguments
- shell tracing
- logs
- deployment metadata
- URLs
- error messages
- generated artifacts
- environment dumps
- debug output

Be particularly careful with:

    set -x

and commands whose arguments may become visible through process inspection.

A release mechanism that leaks credentials is itself an operational failure.

---

# 33. Concurrency

Ask what happens if two deployment processes run concurrently.

Consider:

- same target
- overlapping resources
- different versions
- deployment and rollback simultaneously
- two rollbacks simultaneously
- CI and manual deployment
- automated controller and human action
- agent and human action

Use explicit coordination where required.

A simple lock is not automatically sufficient.

Define:

- scope
- ownership
- lifetime
- stale-lock behavior
- failure behavior

---

# 34. Deployment Ordering

Distributed systems frequently have dependencies.

Do not assume arbitrary order is safe.

Model dependencies explicitly:

    infrastructure
        ↓
    schema
        ↓
    backend
        ↓
    configuration
        ↓
    traffic
        ↓
    cleanup

The actual order depends on the system.

The key principle:

> Deployment ordering must preserve system compatibility at every intermediate
> state.

---

# 35. Intermediate States Matter

A deployment is not only:

    before

and:

    after

There may be many intermediate states.

Ask:

> Is every state that can exist during deployment safe enough?

Examples:

- half-updated fleet
- mixed application versions
- new schema with old application
- new application with old schema
- new config with old code
- old code with new dependency
- one region ahead of another

If an intermediate state is unsafe, redesign the rollout.

---

# 36. Destructive Changes

Treat destruction differently from creation.

Examples:

- delete infrastructure
- remove database fields
- revoke credentials
- delete queues
- remove data
- disable service
- remove feature flags
- decommission hosts

The asymmetry matters:

    turn-up failure
        often creates unused capacity

    turn-down failure
        can destroy active capacity

For destructive operations, require stronger target identity,
confirmation, containment, and verification.

Never interpret:

    empty target set

as automatically meaning:

    delete everything

unless that behavior is explicitly intended and independently guarded.

A `for` loop over a genuinely empty variable is not itself dangerous — it
correctly does nothing, and treating that as a bug in isolation is crying
wolf. The real danger sits one step earlier, at the discovery/selection
call, and it takes a specific shape worth naming: distinguish **confirmed
empty** (the discovery call succeeded and authoritatively reports zero
matching resources) from **empty because the call failed** (an
authentication error, a timeout, an unexpected response shape, or a
malformed filter that silently degrades to zero results). Both can look
identical to the code that consumes the result — an empty list — while
meaning opposite things. Fail-closed logic that no-ops on an empty list is
only safe if the discovery step can prove the emptiness is confirmed
rather than assumed. If discovery cannot distinguish the two, treat an
empty result as UNKNOWN rather than as a safe signal to proceed with
"nothing to do," and require an explicit, successfully-authenticated,
non-error response before trusting zero as a real answer.

---

