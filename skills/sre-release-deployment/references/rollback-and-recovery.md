# sre-release-deployment — Reference: Rollback, Recovery, and Migrations (sections 17-24)

This file is loaded on demand from `sre-release-deployment/SKILL.md` §1a, whenever a rollback, roll-forward, or database migration is involved. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-release-deployment's own core has
already been read.

---

# 17. Retry Safety

Before retrying a failed deployment operation, determine whether it is:

- idempotent
- safely repeatable
- transactional
- partially applied
- externally observable
- reversible

Never blindly retry a non-idempotent operation merely because the client
did not receive a response.

Examples of dangerous retries:

- create resource
- charge payment
- delete resource
- migrate schema
- rotate credential
- increment counter
- trigger external side effect

Prefer:

    inspect state
        ↓
    determine whether previous operation applied
        ↓
    reconcile
        ↓
    retry only if safe

---

# 18. Idempotency

A deployment operation should converge safely when repeated.

Ask:

> What happens if this operation runs twice?

Then:

> What happens if it runs twice after partial completion?

And:

> What happens if the second execution sees state produced by the first?

Idempotency is especially important for:

- retries
- controllers
- automation
- recovery
- CI/CD
- interrupted deployments
- agent-driven operations

---

# 19. Rollback Is Not Magic

Never assume rollback is automatically safer than the forward change.

Rollback itself is a production mutation.

It can fail because:

- dependencies changed
- schema changed
- artifact disappeared
- configuration changed
- credentials changed
- old version is incompatible
- data format is no longer backward compatible
- external APIs changed
- rollback artifact is corrupt
- target disappeared

Therefore rollback requires its own:

    preconditions
    mutation
    verification
    observability
    failure handling

---

# 20. Rollback vs Roll-Forward

Choose based on actual system state.

Rollback is appropriate when:

- previous state remains valid
- data is compatible
- recovery artifact exists
- rollback is tested
- blast radius is controlled

Roll-forward may be safer when:

- schema has changed irreversibly
- old version cannot consume current data
- dependencies moved forward
- rollback introduces greater risk
- the fix itself is safer than restoration

Do not make "always rollback" a dogma.

---

# 21. Recovery Must Be Verified

A rollback is not successful because:

    rollback command returned 0

It is successful when:

    expected previous state was restored
    AND
    postconditions were verified

For example:

    artifact restored
    → hash verified

    configuration restored
    → GET/read-back verified

    resource removed
    → absence verified

Recovery needs the same rigor as deployment.

---

# 22. Recovery Is a Separate Change

Treat:

    deployment

and:

    rollback

as two related but distinct change operations.

Each should have:

- identity
- actor
- timestamp
- state
- target
- result
- evidence
- verification

This prevents recovery from becoming an invisible side effect.

---

# 23. Database Migrations

Database changes require special treatment.

Ask:

- Is the migration backward compatible?
- Can old application versions run against the new schema?
- Can the new application run against the old schema?
- Is the migration reversible?
- Is rollback actually possible?
- Is data transformation destructive?
- Can the migration be interrupted?
- Is it idempotent?
- How is progress measured?
- What happens if only 50% completes?

Never label an irreversible data transformation as "rollbackable"
without defining what rollback actually means.

---

# 24. Expand / Contract

For schema evolution, prefer compatibility-oriented transitions when
appropriate:

    expand
      ↓
    deploy compatible readers/writers
      ↓
    migrate
      ↓
    verify
      ↓
    contract

Avoid coupling an irreversible schema change and application rollout into
one unobservable mutation whenever possible.

---

