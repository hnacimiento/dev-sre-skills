# sre-documentation — Reference: Audience, README, Scope, and Failure/Recovery Documentation (sections 4-10)

This file is loaded on demand from `sre-documentation/SKILL.md`'s
load-on-demand index, when writing or reviewing a README, or documenting what changes, blast radius, failure states, or recovery. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-documentation's own core has already been read.

---

# Audience and Information Architecture

## 4. Documentation Has Different Audiences

A system may have:

- end users
- operators
- developers
- maintainers
- security reviewers
- incident responders
- release engineers
- platform engineers

Do not force all audiences into one document.

Prefer clear separation.

### README

Answer:

- what is this?
- who is it for?
- what does it change?
- what does it not change?
- prerequisites
- basic usage
- expected result
- failure behavior
- recovery
- limitations

### Architecture Documentation

Explain:

- boundaries
- components
- state
- lifecycle
- dependencies
- mutation points
- concurrency
- control loops
- security
- recovery
- verification
- tradeoffs
- limitations

### Runbook

Provide executable operational guidance.

### Security Documentation

Explain:

- trust boundaries
- permissions
- credentials
- secrets
- attack surface
- security assumptions
- limitations

### Incident Documentation

Preserve:

- impact
- evidence
- timeline
- decisions
- hypotheses
- recovery
- contributing conditions
- corrective actions
- validation

---

# README

## 5. README Operational Contract

A useful operational README should answer:

1. What is this?
2. Who is it for?
3. What does it change?
4. What does it NOT change?
5. What are the prerequisites?
6. What resources are affected?
7. What are the risks?
8. How is it executed?
9. How do I know it worked?
10. What happens if it fails?
11. How do I recover?
12. What is not automatically verified?
13. What are the known limitations?

Destructive behavior must not be hidden.

Do not optimize the README exclusively for successful execution.

Optimize it for safe operation.

---

# Operational Scope and Blast Radius

## 6. Document What Changes

Explicitly document:

- resources modified
- resources created
- resources deleted
- resources preserved
- external APIs affected
- credentials accessed
- persistent files created
- generated artifacts
- configuration changes
- destructive operations

Prefer concrete boundaries.

Example:

    This operation modifies X.

    It does not modify Y or Z.

Avoid vague statements such as:

    "This is safe."

---

## 7. Document Blast Radius

For high-impact operations document:

- maximum intended scope
- normal scope
- resources selected
- resources excluded
- possible propagation
- dependencies
- external systems affected
- destructive behavior
- containment mechanisms

If the system supports blast-radius controls, document them.

Examples:

- allowlists
- namespace restrictions
- environment restrictions
- region restrictions
- percentage limits
- approval gates
- dry-run
- canaries
- concurrency limits
- rate limits

Documentation should make the safety boundary visible.

---

# Failure and Recovery

## 8. Document Failure States

Do not document only the happy path.

Document meaningful states such as:

- preflight failure
- validation failure
- dependency failure
- mutation failure
- partial success
- degraded success
- interrupted operation
- timeout
- credential failure
- external API failure
- verification failure
- partial recovery
- failed recovery
- unknown state

If the implementation exposes these states, documentation should explain their meaning.

---

## 9. Recovery Is Part of the Contract

Recovery documentation should answer:

- when recovery should be used
- when recovery should NOT be used
- what prerequisites are required
- what artifacts are required
- where recovery artifacts are stored
- what credentials are required
- what state is restored
- how recovery reports success
- what partial recovery means
- what happens if recovery fails
- how to verify recovery
- when to escalate

Avoid:

    Run rollback.sh.

Prefer:

    Verify the affected state.
    Confirm the required recovery artifact exists.
    Execute recovery.
    Verify the resulting state.
    If verification fails, classify the result as PARTIAL or FAILED and escalate.

Recovery instructions must not depend on guesswork.

---

## 10. Exit Codes Are Not State

Document exact exit-code semantics.

For example:

    0 = operation completed and postconditions verified

    1 = operation failed or recovery is required

    2 = operation completed with a known degraded result

Only use semantics actually implemented.

Never imply:

    exit 0 == system healthy

unless the implementation genuinely verifies that property.

Prefer explicit distinctions between:

- command execution success
- mutation success
- verification success
- system recovery

---

