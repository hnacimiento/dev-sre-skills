# sre-documentation — Reference: Verification, State, and Architecture Documentation (sections 11-20)

This file is loaded on demand from `sre-documentation/SKILL.md`'s
load-on-demand index, when documenting verification scope, state models, architecture boundaries/control loops, or automation. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-documentation's own core has already been read.

---

# Verification

## 11. Document Verification Scope

Always distinguish:

### VERIFIED AUTOMATICALLY

What the system actually checks.

### NOT VERIFIED AUTOMATICALLY

What requires external or human validation.

Example:

    File integrity is verified automatically.

    User-visible application behavior is not verified automatically.

This distinction prevents false confidence.

---

## 12. Document Preconditions and Postconditions

For important operations document:

### Preconditions

What must be true before execution.

### Action

What the system changes.

### Postconditions

What must be true after successful completion.

### Failure State

What may remain after partial failure.

### Recovery

How the system returns to a known state.

Do not describe only commands.

Describe state transitions.

---

# State and Lifecycle

## 13. Use State Models When Behavior Is Complex

When lifecycle behavior is non-trivial, document explicit states.

Example:

    PRECHECK
        ↓
    STAGED
        ↓
    MUTATING
        ↓
    VERIFYING
        ↓
    SUCCESS

Failure:

    MUTATING
        ↓
    RECOVERY
        ↓
    PARTIAL / FAILED / SUCCESS

State models are often clearer than paragraphs.

Document:

- valid transitions
- invalid transitions
- terminal states
- retry behavior
- recovery behavior
- unknown states

---

## 14. Document Operational Invariants

If reliability depends on invariants, document them explicitly.

Examples:

- destructive operations require validated targets
- recovery must report actual recovery state
- successful restore requires verification
- secrets must not appear in generated artifacts
- locks must cover all relevant shared mutations
- an operation must not exceed its blast-radius limit
- an authorization decision cannot be bypassed by the execution path

Then connect important invariants to tests or validation.

Remember:

    documented invariant != guaranteed system safety

The invariant is a local control.

Its interaction with the rest of the system must still be understood.

---

# Architecture Documentation

## 15. Document Boundaries

Architecture documentation should make clear:

- system boundaries
- component boundaries
- trust boundaries
- ownership boundaries
- data boundaries
- mutation boundaries
- recovery boundaries

An engineer should be able to understand where control enters and leaves the system.

---

## 16. Document Control Loops

When automation changes system state, document:

- controller
- controlled system
- observed signal
- control action
- feedback
- delay
- limits
- activation conditions
- inhibition conditions
- retry behavior
- interaction with other controllers

Do not assume independent controllers remain independent in production.

If multiple automated systems can mutate the same resource, document their interaction.

---

## 17. Local Correctness Is Not Global Safety

Avoid documentation that implies:

    component passed tests
    therefore system is safe

A component may:

- pass all tests
- respect its local contract
- return exit 0
- satisfy local invariants

and still contribute to an unsafe global state.

Architecture documentation should identify important interactions and assumptions.

Where appropriate, document:

- races
- feedback loops
- delayed signals
- concurrent actions
- intermediate states
- retries
- conflicting controllers
- stale state
- eventual consistency
- external dependencies

---

# Automation Documentation

## 18. Automation Changes Where Complexity Lives

Document not only what automation removes, but what it introduces.

For significant automation ask:

    What complexity did this remove?

and:

    Where did the complexity move?

Possible consequences:

- simpler execution
- harder diagnosis
- hidden state
- new dependencies
- increased coupling
- new failure modes
- more difficult recovery
- increased cognitive load during incidents

Do not equate:

    more automation

with:

    more reliability

---

## 19. Document Human Interaction With Automation

For operational automation document:

- what the operator sees
- what the operator controls
- what the automation controls
- what actions require confirmation
- what can happen automatically
- how automation can be stopped
- how automation reports progress
- how automation reports uncertainty
- how automation reports failure

Avoid opaque automation.

A useful operational interface should help an operator build an accurate mental model of what the system is doing.

---

## 20. Document Dry-Run Semantics

If dry-run exists, document precisely:

- what is simulated
- what is actually executed
- what external calls occur
- what state is read
- what state is not changed
- whether outputs exactly represent future execution
- known differences between dry-run and real execution

Do not call something "dry-run" if it performs meaningful mutations.

---

