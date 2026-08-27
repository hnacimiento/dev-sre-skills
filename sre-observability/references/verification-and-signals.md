# sre-observability — Reference: Verification, Alerting, and Golden Signals (sections 17-30)

This file is loaded on demand from `sre-observability/SKILL.md` §2a, whenever recovery, alerting, SLOs, or verification telemetry are in question. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-observability's own core has
already been read.

---

# 17. Recovery Must Be Observable

Recovery operations require at least the same observability discipline as
forward operations.

For every recovery resource, determine:

- what was expected
- what was attempted
- what happened
- what was verified
- what failed
- what remains uncertain

A rollback that says "finished" without proving recovery is operationally
dangerous.

---

# 18. Verification Is a Telemetry Producer

Postconditions are observations.

Examples:

### File restore

    exists
    non-empty
    expected hash

### API mutation

    request accepted
    expected state observed through GET

### Deployment

    intended version
    actual version
    health confirmed

### Configuration

    configuration submitted
    configuration retrieved
    semantic equality confirmed

### Resource deletion

    deletion requested
    resource absent
    absence independently verified

Do not treat verification as optional decoration.

---

# 19. Exit Codes and Observability

Exit status is a signal, not a complete observability system.

A useful contract might distinguish:

    0 = verified success
    1 = failure requiring attention
    2 = explicitly defined degraded condition

But the exit code must be backed by observable evidence.

Never infer:

    exit 0 → everything is healthy

without knowing the contract.

Likewise:

    exit 1

does not explain:

- what failed
- where
- why
- what was recovered
- what remains broken

Exit status should be the smallest operational signal, not the entire diagnostic
interface.

---

# 20. Alerts Must Represent User-Relevant Failure

Do not alert merely because something is interesting.

Good alerts generally indicate:

- an SLO is threatened or violated
- an important invariant is violated
- an unsafe condition exists
- human intervention is required
- an automated recovery failed
- a system is approaching an operational limit

Avoid alerts that require an operator to perform meaningless inspection.

Alerting should reduce toil rather than manufacture it.

---

# 21. Symptoms vs Causes

Do not build alerting around assumptions that a symptom identifies its cause.

Example:

    CPU > 90%

is a symptom.

It may result from:

- legitimate traffic
- runaway process
- dependency failure
- retry storm
- memory pressure
- deployment regression

Observability should allow the operator to investigate the causal chain rather
than encode an unverified root cause into the alert.

---

# 22. SLO-Oriented Observability

Observability should connect to reliability objectives.

Ask:

- What user-visible behavior matters?
- Which measurements represent that behavior?
- What constitutes success?
- What constitutes degradation?
- What constitutes an incident?
- Can the telemetry distinguish user impact from internal noise?

Do not optimize observability around whatever is easiest to measure.

Measure what supports reliability decisions.

---

# 23. Golden Signals Are Not the Entire Model

Latency, traffic, errors, and saturation are useful.

They are not sufficient for every system.

Also consider:

- correctness
- integrity
- freshness
- consistency
- completeness
- recovery state
- dependency health
- queue/backlog state
- configuration drift
- rollout state
- authorization failures
- security events

For stateful or transactional systems, correctness can be more important than
availability.

A system returning incorrect data quickly is not necessarily healthy.

---

# 24. Integrity Is Observable State

Whenever integrity matters, observe it explicitly.

Examples:

- checksums
- version identity
- schema version
- configuration hash
- artifact provenance
- expected resource count
- expected membership
- semantic equality

Do not assume:

    HTTP 200
    process alive
    file exists

means:

    correct.

---

# 25. Freshness and Staleness

Telemetry itself has state.

Always consider:

- timestamp
- collection delay
- cache age
- scrape failure
- missing samples
- stale dashboards
- stale configuration
- stale authorization
- stale deployment information

A green dashboard based on stale data is not evidence of health.

When freshness matters, expose freshness.

---

# 26. Observability of Verification

Verification can fail independently from the operation.

Example:

    POST succeeds
    GET verification times out

This does not necessarily mean:

    POST failed

It means:

    postcondition unknown

Do not silently classify unknown verification as failure of the original action
unless that is explicitly the desired safety semantics.

Likewise, do not classify it as success.

Expose the distinction.

---

# 27. Retry Semantics

Retries affect observability.

For an operation with:

    attempt 1 → timeout
    attempt 2 → success

do not expose only:

    SUCCESS

if understanding the transient failure matters.

Record:

- attempts
- retry reason
- final outcome
- elapsed time
- whether retries changed the state

Retries can also create duplicate mutations.

Observability should make this visible.

---

# 28. Idempotency and Observability

Repeated execution should be distinguishable from new mutation where that matters.

Useful information:

- operation ID
- attempt number
- previous state
- desired state
- whether mutation was necessary
- whether the operation converged without mutation

A second successful run should not necessarily look identical to the first.

---

# 29. Concurrency

Observability must expose concurrent actors when concurrency is possible.

Record:

- operation ID
- target
- lock ownership where relevant
- conflicting operation
- acquisition result
- wait/fail-fast behavior

For race conditions, logs should make the sequence reconstructable.

---

# 30. Security Observability

Security events should be observable without leaking secrets.

Consider recording:

- actor identity
- authorization decision
- target
- requested action
- policy result
- approval identity
- approval timestamp
- execution result

Do not record the credential itself.

A useful security event answers:

> Who attempted what against which target, under which authorization, and what
> happened?

---

