# sre-documentation — Reference: Agentic SRE, Security, and Idempotency Documentation (sections 21-29)

This file is loaded on demand from `sre-documentation/SKILL.md`'s
load-on-demand index, when documenting an AI agent's role, security claims/secrets/trust boundaries, or idempotency claims. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-documentation's own core has already been read.

---

# Agentic SRE Documentation

## 21. Agents Are Part of the System

If an AI agent participates in operations, document it as an operational component.

Document:

- inputs
- context
- tools
- permissions
- policies
- decisions
- actions
- outputs
- feedback
- supervision
- uncertainty
- failure behavior
- audit trail

Do not document an agent as:

    AI magically fixes incidents.

Describe its actual control boundary.

---

## 22. Separate Reasoning From Authority

For agentic operations distinguish:

    OBSERVATION
        ↓
    INTERPRETATION
        ↓
    RECOMMENDATION
        ↓
    AUTHORIZATION
        ↓
    EXECUTION
        ↓
    VERIFICATION

Document which layer is responsible for each step.

An agent may have autonomous reasoning without having unlimited mutation authority.

---

## 23. Deterministic Boundaries

Use deterministic mechanisms for properties requiring deterministic enforcement where practical.

Examples:

- authorization
- schema validation
- allowlists
- resource limits
- blast-radius limits
- access control
- cryptographic operations
- hashing
- structured parsing
- critical calculations
- policy enforcement
- invariant enforcement

Use probabilistic reasoning where it provides useful adaptation:

- interpretation
- synthesis
- correlation
- hypothesis generation
- exploration
- summarization
- option generation

Do not document:

    "The model will always behave correctly."

Document enforceable boundaries instead.

Remember:

    deterministic component != globally safe system

The word chosen to describe what a component does is itself a claim that
needs evidence, separate from whether the underlying behavior is good.
Verbs like "validates," "enforces," "verifies," or "blocks" imply a
deterministic boundary exists behind them — something that can say no and
be relied on to say no consistently. If the actual mechanism is a model
reasoning in natural language about whether an action seems permitted,
with no policy engine capable of refusing it, then writing "the agent
validates deployment permissions" is a misleading claim regardless of how
well the model happens to perform in practice. Prefer verbs that match
the actual mechanism: "the agent assesses," "the agent flags," "the agent
recommends" for reasoning-only behavior, reserving "validates,"
"enforces," and "blocks" for a boundary that is actually deterministic
and testable. This is a distinct check from the general "avoid unsupported
words" rule in section 26 — it applies even when nobody used a dramatic
word like "secure" or "guaranteed," because an ordinary-sounding verb can
carry the same false promise of enforcement.

---

## 24. Agent Security Documentation

If an agent can access production systems, document:

- authentication
- authorization
- tool permissions
- credential scope
- prompt/context sources
- untrusted inputs
- external content
- prompt injection exposure
- tool injection risks
- auditability
- approval mechanisms
- action limits
- kill switch or disable mechanism
- fallback behavior

Do not assume human-in-the-loop eliminates all risk.

The human must receive sufficient evidence and context to make a meaningful authorization decision.

---

## 25. Agent Verification

Document separately:

### What the agent can reason about

and:

### What the surrounding deterministic system verifies

Examples:

    Agent:
    proposes remediation.

    Policy:
    validates allowed action.

    Authorization:
    confirms authority.

    Execution layer:
    performs only permitted mutation.

    Verification:
    checks resulting system state.

The model's confidence is not equivalent to system verification.

---

# Security Documentation

## 26. Security Claims Require Strong Evidence

Avoid unsupported words:

    secure
    safe
    impossible
    guaranteed
    never leaks
    cannot fail

unless the scope is explicitly defined and demonstrated.

Prefer precise claims.

Example:

    The deployment process checks that the configured API key value is absent from generated JavaScript artifacts before deployment.

Specific claims are more trustworthy than broad claims.

---

## 27. Document Credential and Secret Lifecycle

Document:

- where credentials originate
- where they are stored
- how they are passed
- who can access them
- whether they persist
- when they expire
- whether they appear in process arguments
- whether they appear in logs
- whether generated artifacts can contain them
- cleanup behavior
- unavoidable exposure windows

Do not claim:

    credentials are never exposed

if the implementation has any exposure window.

State the actual boundary.

---

## 28. Document Trust Boundaries

For security-sensitive systems document:

- trusted inputs
- untrusted inputs
- trust transitions
- privileged operations
- authorization boundaries
- external dependencies
- generated artifacts
- operator access
- agent access

Make security assumptions explicit.

---

# Idempotency and Repeatability

## 29. Do Not Casually Claim Idempotency

If documentation says:

    safe to run multiple times

define what that means.

Document:

- repeated execution behavior
- state after first execution
- state after second execution
- whether backups are regenerated
- whether state is recalculated
- whether external side effects repeat
- behavior after partial failure
- behavior after interruption

"Idempotent" is a technical property, not a synonym for "probably safe."

---

