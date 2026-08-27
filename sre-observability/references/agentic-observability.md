# sre-observability — Reference: Agentic Observability (sections 31-42)

This file is loaded on demand from `sre-observability/SKILL.md` §2a, whenever an AI agent's actions, decisions, or outputs need to be observable. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-observability's own core has
already been read.

---

# 31. Agentic SRE Observability

If an AI agent participates in operations, the agent becomes part of the
observable control loop.

Observe at least:

### Context

- incident/request identifier
- relevant environment
- data sources consulted
- freshness of data

### Reasoning boundary

Do not depend on hidden chain-of-thought.

Instead record structured operational rationale such as:

- hypothesis
- evidence references
- confidence where appropriate
- proposed action
- reason action is permitted

### Tool usage

Record:

- tool
- operation
- target
- authorization result
- execution result
- verification result

### Human interaction

Record:

- proposed action
- approval/rejection
- approving identity
- approval scope
- whether the target changed before execution

### Safety

Record:

- blocked actions
- policy violations
- denied tools
- suspicious inputs
- prompt injection detections
- policy boundary crossings

The goal is to make the agent's operational behavior auditable without treating
the model's internal reasoning as a trustworthy security boundary.

---

# 32. Agent Output Is Not Evidence

Never treat:

    "Rollback completed successfully."

from an agent as evidence.

The evidence must come from deterministic observations such as:

- resource state
- API GET
- checksum
- deployment version
- health signal
- authorization record

The agent can interpret evidence.

It should not manufacture the evidence that proves its own success.

---

# 33. Agent Decision vs Agent Action

Distinguish:

    PROPOSED
    APPROVED
    EXECUTED
    VERIFIED

Example:

    rollback
      PROPOSED
      ↓
      APPROVED
      ↓
      EXECUTED
      ↓
      VERIFIED

Do not collapse these into:

    AGENT_SUCCESS

This distinction is essential for incident reconstruction.

---

# 34. Prompt Injection Observability

Treat external content consumed by an agent as potentially untrusted.

Examples:

- logs
- tickets
- comments
- dashboards
- deployment metadata
- documentation
- repository files
- HTTP responses

Observe:

- suspicious instruction patterns
- attempted tool manipulation
- policy violations
- blocked actions
- source of the untrusted content

Do not assume the model can reliably distinguish data from instructions without
architectural support.

---

# 35. Tool-Level Observability for Agents

Prefer observable structured tools over arbitrary shell execution.

A useful event:

    tool = restart_service
    target = production-api
    authorization = approved
    execution = success
    verification = healthy

is superior to:

    tool = execute_shell
    command = "..."

Structured tools improve:

- auditability
- authorization
- blast-radius analysis
- incident reconstruction
- policy enforcement

---

# 36. Deterministic Verification

Whenever a property can be verified deterministically, prefer deterministic
verification.

Examples:

- SHA256
- exact version
- schema validation
- resource existence
- HTTP status
- semantic JSON comparison
- authorization result

Do not ask an LLM:

> "Does this hash look correct?"

Use the hashing algorithm.

Do not ask an LLM:

> "Does this JSON contain the expected field?"

Use deterministic parsing.

The agent may interpret the result, but should not replace the mechanism that
provides the guarantee.

---

# 37. System-Level Observability

Component telemetry is insufficient.

A system can have:

- healthy database
- healthy API
- healthy worker
- healthy queue
- healthy deployment

and still be broken because their interaction violates a system-level
invariant.

Therefore define and observe system-level properties.

Examples:

    request accepted
        →
    state persisted
        →
    state propagated
        →
    response reflects persisted state

or:

    backup created
        →
    mutation performed
        →
    verification passed
        →
    rollback artifact available

Observe the chain, not just its components.

---

# 38. Invariants

Define important invariants explicitly.

Examples:

- no secret appears in deployed artifacts
- every successful restore has a matching verified hash
- every destructive operation has an authorized target
- every recovery attempt produces a terminal result
- every generated executable artifact passes syntax validation
- every successful deployment has a verified expected version
- every agent mutation has an authorization record
- no operation claims success without required postcondition evidence

Then instrument the system to detect invariant violations.

---

# 39. Missing Telemetry Is Itself a Signal

Examples:

- no heartbeat
- no metrics
- no deployment event
- no recovery result
- no verification event
- no audit record

Do not automatically interpret missing telemetry as "nothing happened".

It may mean:

    telemetry failed
    process died
    collector failed
    operation aborted
    network partition
    state became inaccessible

Missing expected telemetry should be detectable where practical.

---

# 40. Cardinality and Cost

Observability itself has operational cost.

Watch for:

- unbounded labels
- user IDs as metric labels
- request IDs as metric dimensions
- arbitrary URLs
- exception text as metric labels
- high-cardinality resource identifiers

Use logs/traces for high-cardinality detail where appropriate.

Do not solve observability by creating an observability outage.

---

# 41. Retention and Incident Reconstruction

Telemetry should survive long enough to answer operational questions.

Consider:

- retention
- clock synchronization
- timestamp precision
- correlation identifiers
- artifact retention
- deployment history
- configuration history
- recovery artifacts

For important operations, ensure the evidence remains available after the
process exits.

---

# 42. Time and Clock Problems

Distributed systems have imperfect clocks.

Do not assume timestamp ordering alone proves causal ordering.

When causal ordering matters, use:

- operation IDs
- request IDs
- sequence numbers
- trace relationships
- explicit state transitions

Wall-clock timestamps are useful context, not absolute causality.

---

