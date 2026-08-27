# sre-observability — Reference: The Control Loop and Observable State (sections 3-16)

This file is loaded on demand from `sre-observability/SKILL.md` §2a, whenever you need to decide what telemetry a system emits about its own state. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-observability's own core has
already been read.

---

# 3. Observability Is About Questions

Before choosing telemetry, identify the operational questions the system must
be able to answer.

At minimum:

### State

- What state is the system in?
- What state does it believe it is in?
- What state was actually observed?

### Transition

- What changed?
- When did it change?
- Who or what caused the change?
- What was the previous state?

### Causality

- What operation preceded the failure?
- Which dependency failed?
- Which decision caused the mutation?
- What evidence supports the suspected cause?

### Scope

- What resources were affected?
- What resources were not affected?
- What is the current blast radius?

### Recovery

- Was recovery attempted?
- Which resources were recovered?
- Which were not?
- Was recovery verified?
- What remains uncertain?

### Safety

- Was an authorization check performed?
- Was the target verified?
- Was a destructive operation attempted?
- Was a safeguard triggered?
- Was a safeguard bypassed?

### Human operation

- What did the operator see?
- What decision did they make?
- What information was available at that moment?
- What information was missing?

### Automation

- What did the automation attempt?
- What assumptions did it make?
- Which guardrails were applied?
- Which actions were blocked?
- Which actions succeeded only partially?

### Agentic operation

- What context did the agent receive?
- What tools did it invoke?
- What authorization did it possess?
- What action did it propose?
- What action was actually executed?
- What evidence justified execution?
- Did the environment change between decision and execution?

---

# 4. Observability Is a Control-Loop Property

Do not design telemetry independently from the action being controlled.

For every meaningful operational action, reason about:

    OBSERVE
       ↓
    INTERPRET
       ↓
    DECIDE
       ↓
    ACT
       ↓
    VERIFY

Every stage can fail.

Examples:

### Observation failure

The system is unhealthy but telemetry is missing.

### Interpretation failure

Telemetry exists but is misleading or ambiguous.

### Decision failure

The operator or automation interprets correct telemetry incorrectly.

### Action failure

The chosen operation does not execute correctly.

### Verification failure

The operation succeeds but the verification path is broken.

The last case is particularly dangerous because it can produce false confidence.

---

# 5. Desired State vs Observed State

Always distinguish:

- desired state
- intended state
- reported state
- observed state
- verified state
- unknown state

Do not collapse them into one status variable.

Example:

    desired = RESTORED
    reported = SUCCESS
    observed = FILE_EXISTS
    verified = HASH_MATCH
    API_STATE = UNKNOWN

The system is not simply "SUCCESS".

Its actual operational state is partially known.

A good observable system makes uncertainty explicit.

---

# 6. Never Hide Unknown

Unknown is a legitimate operational state.

Do not convert:

- timeout
- missing telemetry
- unavailable API
- failed verification
- interrupted operation
- missing state
- stale cache
- unavailable dependency

into:

- SUCCESS
- HEALTHY
- NOT_NEEDED
- NO_CHANGE
- DEGRADED

unless the semantics genuinely justify that classification.

Prefer:

    VERIFIED
    FAILED
    DEGRADED
    PARTIAL
    UNKNOWN
    NOT_ATTEMPTED
    NOT_APPLICABLE

over optimistic binary states when the operation requires more nuance.

---

# 7. Status Must Have Semantics

Every status exposed by a system should answer:

1. What does it mean?
2. What evidence produces it?
3. What evidence disproves it?
4. What remains unknown?
5. What operator action should follow?

For example:

    ROLLBACK_SUCCESS

should mean something stronger than:

    rollback() returned 0

A useful semantic contract might be:

    ROLLBACK_SUCCESS =
        every resource in scope
        has a successful action outcome
        AND
        every required postcondition was verified

If that contract cannot be demonstrated, the status must not claim success.

---

# 8. Telemetry Types

Use the appropriate signal for the question.

## Metrics

Best for:

- rates
- counts
- distributions
- saturation
- latency
- resource utilization
- SLO calculations
- fleet-wide trends

Metrics are weak at explaining individual causal sequences.

## Logs

Best for:

- discrete events
- decisions
- transitions
- failures
- operator actions
- security events
- detailed context

Logs should be structured where practical.

## Traces

Best for:

- distributed request paths
- causal relationships across services
- latency attribution
- dependency interactions

## State snapshots

Critical for:

- recovery
- deployment state
- configuration state
- resource ownership
- reconciliation
- incident reconstruction

## Artifacts

Examples:

- manifests
- checksums
- backup metadata
- generated rollback scripts
- deployment records
- authorization records

Artifacts can provide durable evidence that ephemeral telemetry cannot.

## 8a. Six Related Terms That Are Not Synonyms

These terms get used interchangeably in casual conversation. They are not
interchangeable, and confusing them is itself an observability failure
mode. Each answers a different question and depends on the ones before it.

**Monitoring** answers "is a known signal outside an expected range right
now?" It is passive, threshold-based, and only as good as the signals
someone thought to collect in advance.

**Observability** answers "can I determine the system's actual internal
state from what it exposes externally, including for a question nobody
anticipated?" It is the system property that monitoring draws on; a
system can have extensive monitoring and still be poorly observable if
none of its signals can answer the specific question this incident raised.

**Verification** answers a narrower, action-scoped question: "did this
specific mutation actually produce the specific postcondition it was
supposed to produce?" It is observability applied at one mutation
boundary, immediately after that mutation, and it is what section 2 of
this skill is built around.

**Auditability** answers "who did what, under what authorization, to
which target, and when?" It exists for accountability and reconstruction
after the fact, and it must hold even when nothing failed — a successful
operation still needs an audit trail.

**Diagnostics** is the active practice of using observability, logs, and
traces to locate a root cause once a problem is already suspected. It
consumes what monitoring, observability, and auditability produced; it
does not itself produce new telemetry.

**Incident reconstruction** is the highest-level synthesis: assembling
monitoring signals, observability data, verification results, audit
records, and diagnostic findings into a single coherent timeline after
the fact (see section 57). It is only as complete as the weakest link
beneath it — a gap in verification or a missing audit record becomes a
gap in the reconstructed story no matter how much monitoring existed.

A system can score well on one of these and poorly on another. Extensive
monitoring with no verification will alert on symptoms but cannot confirm
a fix worked. Strong verification with no auditability can confirm state
but cannot answer who changed it. Treat these as six distinct properties
to evaluate separately, not as one undifferentiated "observability"
checkbox.

---

# 9. Logs Are Evidence, Not Truth

Never assume a log message is proof of the state it describes.

For example:

    echo "Rollback successful"

is an assertion.

A stronger design records:

- operation
- target
- resource
- action
- expected state
- observed state
- verification result
- evidence
- timestamp
- correlation/run identifier

The log should make it possible to reconstruct what happened.

---

# 10. Structured Events

Prefer events that are machine-readable and semantically explicit.

A useful event conceptually contains:

    timestamp
    operation_id
    operation
    phase
    actor
    target
    resource
    action
    expected_state
    observed_state
    verification
    result
    error
    correlation_id

Do not blindly add every field everywhere.

The principle is:

> Include enough context to reconstruct the operational decision and outcome.

---

# 11. Do Not Log Secrets

Observability must never become a secret-exfiltration mechanism.

Do not expose:

- API keys
- passwords
- bearer tokens
- private keys
- session credentials
- sensitive configuration
- authentication headers
- secret material embedded in payloads

Be especially careful with:

- command logging
- shell tracing
- debug output
- exception dumps
- HTTP headers
- generated scripts
- subprocess arguments
- container exec commands
- agent tool calls

A system can be operationally observable and simultaneously insecure.

That is a design failure.

---

# 12. Logging Level Is Not a Security Boundary

Do not assume:

> "It is only DEBUG, so exposing the secret is acceptable."

Debug logs are frequently:

- enabled during incidents
- collected centrally
- retained longer than expected
- accessible to more people
- copied into tickets
- attached to postmortems
- consumed by automated systems

Sensitive information should be excluded at the source whenever possible.

---

# 13. Correlation and Causality

Operational systems must support reconstruction across boundaries.

Use stable identifiers such as:

- operation ID
- request ID
- deployment ID
- recovery ID
- incident ID
- resource ID

A useful chain looks like:

    incident
       ↓
    operator/agent action
       ↓
    operation
       ↓
    mutation
       ↓
    verification
       ↓
    resulting state

Avoid relying exclusively on timestamps to reconstruct causality.

---

# 14. State Transitions Must Be Observable

For stateful operations, expose transitions.

Example:

    INIT
      ↓
    PRECHECK
      ↓
    STAGED
      ↓
    MUTATING
      ↓
    VERIFYING
      ↓
    SUCCESS

or:

    MUTATING
      ↓
    FAILURE
      ↓
    RECOVERY
      ↓
    PARTIAL

Do not merely log:

    START
    END

The important information is often what happened between them.

---

# 15. Partial Failure Must Be Visible

Distributed and multi-resource operations commonly produce partial success.

Examples:

    5/6 files restored
    2/3 replicas updated
    API succeeded but verification failed
    configuration updated but restart failed

Do not compress this into:

    FAILED

or:

    SUCCESS

when the distinction matters operationally.

Expose:

- attempted
- succeeded
- failed
- not attempted
- verified
- unverified

This is especially important for recovery.

---

# 16. Resource-Level Observability

When an operation touches multiple resources, aggregate status must be derived
from resource-level outcomes.

Example:

    index.html     RESTORED
    addon-1.js     RESTORED
    addon-2.js     ATTEMPT_FAILED
    addon-3.js     NOT_ATTEMPTED
    addon-4.js     RESTORED
    addon-5.js     RESTORED
    addon-6.js     RESTORED
    CustomCss      RESTORED

Aggregate:

    PARTIAL

not:

    SUCCESS

The aggregate must not erase the evidence underneath it.

---

