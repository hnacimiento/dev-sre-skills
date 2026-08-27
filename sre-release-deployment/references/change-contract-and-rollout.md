# sre-release-deployment — Reference: Change Contract and Rollout Verification (sections 2-16)

This file is loaded on demand from `sre-release-deployment/SKILL.md` §1a, before authorizing or reviewing any change, deployment, canary, or rollout. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-release-deployment's own core has
already been read.

---

# 2. Change Is a Reliability Event

Every production mutation has:

- a target
- an intended state
- preconditions
- actions
- dependencies
- possible intermediate states
- failure modes
- verification criteria
- recovery strategy
- blast radius
- ownership
- evidence

Before proposing implementation, identify these explicitly.

If they cannot be identified, the release design is incomplete.

---

# 3. Change Contract

Every meaningful release should have an explicit change contract.

At minimum define:

    WHAT
    WHY
    WHERE
    WHEN
    WHO
    HOW
    EXPECTED STATE
    VERIFICATION
    ABORT CONDITION
    RECOVERY
    FINAL RESULT

A useful conceptual contract is:

    Preconditions
        ↓
    Mutation
        ↓
    Postconditions

Never assume that satisfying the preconditions guarantees the
postconditions.

Never assume that executing the mutation successfully proves the
postconditions.

---

# 4. Preconditions

Before mutation begins, validate assumptions that are required for safety.

Examples:

- target exists
- target identity is correct
- artifact exists
- artifact integrity is valid
- dependencies are compatible
- required capacity exists
- credentials are valid
- authorization exists
- current version is expected
- configuration is compatible
- backup/recovery capability exists
- required observability exists
- no conflicting deployment is active
- environment is the intended environment
- destructive operation has sufficient confirmation

Fail closed when a safety-critical precondition cannot be established.

Do not silently substitute:

    unknown

with:

    probably okay

---

# 5. Preconditions Must Be About the Real Target

A common failure mode is validating one thing and mutating another.

Examples:

- validating container A and deploying to container B
- validating hostname A and connecting to hostname B
- validating artifact version A and deploying version B
- validating one configuration snapshot and modifying another
- trusting stale cached identity

Identity must be resolved and, where appropriate, revalidated immediately
before mutation.

Cached state may be a convenience.

It is not automatically trusted identity.

---

# 6. Artifact Integrity

Before deployment, establish what artifact is actually being deployed.

Prefer:

- immutable version identifiers
- content digests
- checksums
- signed artifacts where appropriate
- provenance
- reproducible build information
- explicit source/version metadata

Avoid relying exclusively on mutable identifiers such as:

    latest
    current
    main
    stable

A mutable identifier can be useful for discovery.

It is not equivalent to immutable identity.

The release system should make it possible to answer:

> Exactly which artifact was deployed?

---

# 7. Artifact Provenance

A production release should preserve enough information to reconstruct:

- source revision
- build identifier
- artifact identifier
- artifact digest
- dependency versions
- configuration version
- deployment tool/version
- deployment actor
- authorization context
- target
- timestamp
- environment
- relevant feature flags
- release policy

Provenance is part of operational evidence.

If the team cannot determine what was deployed, incident analysis becomes
guesswork.

---

# 8. Blast Radius

Before mutation, determine:

- how many resources can change
- which users can be affected
- which regions can be affected
- which services depend on the change
- whether the change can cascade
- whether rollback affects more resources than deployment
- whether the change can propagate automatically
- whether a bad artifact can affect the entire fleet

Prefer mechanisms that constrain blast radius.

Examples:

- one instance
- one host
- one container
- one shard
- one region
- one tenant
- small percentage of traffic
- canary population

Then expand only after verification.

---

# 9. Progressive Delivery

When practical, prefer:

    small exposure
        ↓
    observe
        ↓
    verify
        ↓
    expand

over:

    full fleet mutation
        ↓
    hope

Progressive delivery can include:

- canaries
- staged rollout
- regional rollout
- percentage rollout
- tenant-based rollout
- feature flags
- shadow traffic
- dark launches
- incremental migrations

The purpose is not ceremony.

The purpose is to reduce the number of systems exposed to an unknown failure
before evidence exists that the change is safe.

---

# 10. Canary Is an Experiment

A canary is not merely "deploying to fewer machines."

It is an experiment.

Define beforehand:

- hypothesis
- population
- duration
- signals
- success criteria
- abort criteria
- comparison baseline

Example conceptual structure:

    Hypothesis:
        New version does not materially worsen request latency.

    Population:
        1% of traffic.

    Observation:
        latency, errors, saturation, correctness.

    Abort:
        statistically/significantly worse behavior.

    Decision:
        continue or stop.

Avoid canaries that have no meaningful decision criteria.

---

# 11. Deployment Verification

Verification must be explicit.

Do not treat:

    command succeeded

as:

    deployment succeeded

Verification can include:

- process exists
- expected version is running
- artifact digest matches
- configuration matches expected state
- endpoint responds
- health checks pass
- API behavior is correct
- critical transactions succeed
- dependency connectivity works
- error rate is acceptable
- latency is acceptable
- saturation is acceptable
- business invariants hold
- no unexpected secrets are exposed
- expected resources exist
- unexpected resources do not exist

Prefer verification against the resulting state.

---

# 12. Technical Health vs System Health

A deployment can be technically healthy while the system is unhealthy.

Examples:

- process is running but returns incorrect data
- HTTP 200 but business transaction is wrong
- database migration succeeded but application semantics are broken
- all containers are healthy but traffic routing is wrong
- CPU is normal but correctness is degraded

Therefore distinguish:

    component health
    service health
    system health
    business correctness

Do not infer the latter solely from the former.

---

# 13. Verification Must Be Independent Where Necessary

A deployment should not be allowed to declare itself successful solely based
on its own internal assumptions.

Examples:

Bad:

    deployment command returns 0
    → SUCCESS

Better:

    deployment command returns 0
    → inspect resulting state
    → compare expected version
    → execute health/correctness checks
    → SUCCESS

For critical properties, use deterministic verification mechanisms where
possible.

Examples:

- SHA256
- exact version comparison
- schema validation
- structured API response validation
- explicit state comparison
- deterministic invariants

Do not ask a probabilistic system to provide a guarantee that deterministic
software can provide more reliably.

---

# 14. Rollout State Machine

Represent deployment state explicitly.

A useful conceptual model:

    INIT
      ↓
    PRECHECK
      ↓
    READY
      ↓
    MUTATING
      ↓
    VERIFYING
      ↓
    SUCCESS

Possible failure paths:

    PRECHECK
      ↓
    FAILED_NO_MUTATION

    MUTATING
      ↓
    FAILED_AFTER_MUTATION

    VERIFYING
      ↓
    FAILED_AFTER_MUTATION

After mutation failure:

    RECOVERY
      ↓
    VERIFIED_RECOVERY
    or
    PARTIAL_RECOVERY
    or
    FAILED_RECOVERY

Do not collapse these states into a single boolean.

The distinction between:

    failed before mutation

and:

    failed after partial mutation

is operationally critical.

---

# 15. Partial Deployment

Partial deployment is a first-class state.

Examples:

- 3 of 10 nodes updated
- application updated but configuration not updated
- backend deployed but frontend failed
- database migration completed but application rollout failed
- one region updated and another failed
- some resources restored and others not

Never report:

    SUCCESS

when the system is actually mixed.

Represent partial state explicitly.

A useful result vocabulary may include:

    SUCCESS
    DEGRADED
    PARTIAL
    FAILED
    UNKNOWN

The exact vocabulary can vary by system.

The semantic distinction must not disappear.

---

# 16. UNKNOWN Is Not SUCCESS

If the system cannot establish whether the desired state exists, do not
convert uncertainty into success.

Examples:

- deployment command timed out
- connection lost during mutation
- process died before final verification
- verification endpoint is unavailable
- state cannot be queried
- controller lost communication with target

Possible state:

    UNKNOWN

Then require an explicit reconciliation strategy.

Do not blindly retry a mutation whose first attempt may have succeeded.

A timeout can mean:

    operation failed

or:

    operation succeeded but acknowledgement was lost

Treat this as a distributed-systems problem.

---

