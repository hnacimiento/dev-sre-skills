# sre-testing — Reference: Contract, State, and Failure Testing (sections 2-7)

This file is loaded on demand from `sre-testing/SKILL.md`'s load-on-demand
index, when designing tests for what a component promises, its state machine, and its failure/fault behavior. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-testing's own core has already been read.

---

## 2. Test the contract, not the implementation

Prefer testing externally observable contracts over internal implementation details.

For every meaningful operation identify:

- Preconditions
- Inputs
- State before
- Intended mutation
- Expected state after
- Postconditions
- Observable evidence
- Failure classification
- Recovery behavior
- Exit status
- Artifacts/logs produced

If implementation changes while the contract remains valid, the tests should generally remain valid.

Do not create tests that merely prove that the code currently looks the way it does.

## 3. Failure-first testing

For every mutation ask:

"What can fail immediately after this point?"

Then test that failure.

A useful sequence is:

1. establish initial state
2. execute operation
3. inject failure
4. observe resulting state
5. verify reported result
6. verify exit status
7. verify recovery
8. verify residual artifacts
9. verify operator-facing evidence

Do not stop at "the command returned non-zero."

A command can fail correctly while the system remains inconsistent.

Conversely, a command can return zero while the desired state is not actually achieved.

## 4. State-machine testing

If the system has explicit states, test transitions rather than only endpoints.

For example:

INIT
STAGED
MUTATING
VERIFYING
SUCCESS
DEGRADED
FAILED
RECOVERY
PARTIAL

For each transition identify:

- trigger
- expected state
- forbidden states
- observable evidence
- exit code
- recovery requirement

Test both:

- valid transitions
- invalid transitions

Particularly test transitions caused by unexpected failures.

## 5. Postcondition testing

Never equate:

command succeeded

with:

operation succeeded.

A successful command is only evidence that the command reported success.

The system must verify the postcondition.

Examples:

- file copied → verify existence
- configuration changed → read it back
- file restored → verify hash
- resource removed → verify absence
- API update succeeded → GET the resulting state
- deployment completed → verify deployed artifact
- lock acquired → verify expected ownership semantics
- generated script created → execute syntax validation and representative behavior

When possible:

desired state = observed state

must be explicitly demonstrated.

## 6. Positive and negative testing

Every important feature needs at least:

### Positive tests

Verify the intended behavior succeeds.

### Negative tests

Verify invalid or unsafe conditions fail safely.

### Recovery tests

Verify the system can recover from failure.

### Regression tests

Verify previously discovered bugs remain fixed.

A bug that has once escaped into production should normally become a permanent regression test.

## 7. Fault injection

Fault injection should be deliberate and controlled.

Useful failure classes include:

- command returns non-zero
- command returns zero but produces invalid state
- timeout
- connection failure
- permission denied
- missing file
- corrupted file
- malformed input
- malformed configuration
- unavailable dependency
- dependency disappears during execution
- partial write
- partial copy
- stale state
- stale cache
- unexpected process termination
- SIGINT
- SIGTERM
- SIGKILL
- disk exhaustion
- read-only filesystem
- concurrent execution
- race conditions

Do not test only the failure you already know exists.

Construct failures from the system's assumptions.

