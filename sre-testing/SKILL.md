---
name: sre-testing
description: >
  SRE-focused testing strategy for establishing evidence that a system
  behaves correctly under expected, degraded, failed, concurrent,
  interrupted, and adversarial conditions. Apply when designing or
  reviewing tests for scripts, services, automation, recovery mechanisms,
  generated code, or AI-assisted operations. Covers failure-first and
  postcondition testing, fault injection, state-machine and idempotency
  testing, security and observability testing, and testing AI agents and
  their interactions with the rest of the system.
---

# sre-testing

You are operating with an SRE testing mindset.

Your purpose is not merely to verify that code works.
Your purpose is to establish evidence that a system behaves correctly under expected, degraded, failed, concurrent, interrupted, and adversarial conditions.

Testing is part of the reliability design, not a final activity.

## 1. Core principle

Never ask only:

"Does it work?"

Ask:

"How do we know it works, what evidence proves that, and what happens when every important assumption is false?"

A test suite must therefore establish:

- expected behavior
- failure behavior
- recovery behavior
- observable outcomes
- boundary behavior
- concurrency behavior
- interruption behavior
- security-relevant behavior
- idempotency
- operational usability
- documentation/code consistency

A passing happy-path test is weak evidence.

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

## 8. Bash-specific testing

For Bash automation, test at minimum:

- `set -e` behavior
- `set -u` behavior
- `pipefail`
- traps
- signal handling
- exit codes
- subshell behavior
- command substitution
- pipelines
- functions invoked under `||`
- functions invoked conditionally
- cleanup behavior
- temporary-file handling
- quoting
- filenames containing spaces
- filenames containing shell metacharacters
- empty variables
- missing variables
- empty command output
- unexpected command output
- partial writes
- interrupted writes
- lock acquisition
- lock contention
- generated shell scripts
- heredoc expansion
- runtime versus generation-time variable expansion

Never assume `set -e` provides reliable transactional semantics.

Test the actual control flow.

## 9. Generated-code testing

Generated scripts are independent software artifacts.

If a program generates:

- rollback scripts
- migration scripts
- deployment scripts
- recovery scripts
- configuration files
- shell fragments

test both:

1. the generator
2. the generated artifact

Minimum validation for generated Bash:

- `bash -n`
- ShellCheck when applicable
- representative execution
- failure-path execution
- exit-code verification
- generated content inspection

Do not assume that because the generator passes tests, the generated script is correct.

## 10. Recovery testing

Recovery is a first-class operation.

For every recoverable mutation test:

- recovery succeeds
- recovery partially succeeds
- recovery completely fails
- recovery is interrupted
- recovery dependency is unavailable
- recovery source is missing
- recovery source is corrupted
- target disappears during recovery
- verification after recovery fails

A recovery routine must never claim success that it cannot demonstrate.

If the system exposes:

SUCCESS
PARTIAL
FAILED
NOT_NEEDED

test the distinction between them explicitly.

Do not allow:

"recovery attempted"

to become equivalent to:

"recovery succeeded."

## 11. Resource-level outcomes

For multi-resource operations, test resources independently.

If an operation modifies:

- A
- B
- C
- D

inject failure into B and verify:

- A outcome is reported correctly
- B outcome is reported correctly
- C is still evaluated when safe
- D is still evaluated when safe
- aggregate result is correct

One failed resource must not hide the state of the remaining resources unless continuing would itself be unsafe.

Test for silent omission.

## 12. Idempotency

Run important operations repeatedly.

At minimum:

- first execution
- second execution
- third execution

Verify:

- no duplicated configuration
- no duplicated files
- no unnecessary mutations
- stable resulting state
- stable hashes where applicable
- no accumulation of unintended artifacts
- same meaningful outcome

For recovery operations, test repeated recovery as well.

Ask:

"What happens if the operator reasonably retries this command?"

Retries are normal operational behavior.

## 13. Concurrency

Assume operators and automation can overlap.

Test:

- same target concurrently
- different targets concurrently
- lock contention
- lock release after failure
- lock release after SIGTERM
- lock release after process death
- shared-state races
- atomic versus non-atomic writes
- stale state reads

A lock is not automatically proof of concurrency safety.

Test the scope of the lock.

Test what remains outside the lock.

## 14. Interruption testing

Interrupt operations at meaningful mutation boundaries.

Test:

- before mutation
- during mutation
- after partial mutation
- during verification
- during rollback
- during cleanup
- during state-file writes

For each interruption determine:

- what state remains
- whether recovery is possible
- whether recovery artifacts exist
- whether the exit status is meaningful
- whether the operator knows what happened

SIGKILL and power loss cannot be trapped.

Therefore test the artifacts and state that must make manual recovery possible afterward.

## 15. Security testing

Security controls must be tested against the actual threat they claim to prevent.

Examples:

If the contract says:

"secret must not appear in artifact"

test:

- secret absent
- secret present
- secret-like placeholder present
- variable name present
- empty secret
- malformed secret
- secret passed through command arguments
- secret exposed in logs
- secret exposed in generated files

Do not test only the string representation of the variable.

Test the actual sensitive value and its possible exposure paths.

## 16. Observability testing

A failure that cannot be diagnosed is operationally expensive.

Tests should verify that failures produce enough evidence to answer:

- What operation was running?
- Which resource failed?
- Why did it fail?
- What had already changed?
- What was recovered?
- What was not recovered?
- What should the operator do next?
- What is the exit status?
- Where are recovery artifacts?
- Can the result be distinguished from success?

Test logs and summaries as part of the operational contract.

Do not require operators to reconstruct state from raw debug output.

## 17. Exit-code testing

Exit codes are part of the API of automation.

For every meaningful result test:

- stdout/stderr
- exit code
- resulting state

Never test exit code independently of system state.

Examples:

SUCCESS should not merely mean:

"last command returned 0."

DEGRADED should only be used when degradation is known and explicitly characterized.

PARTIAL recovery must not silently become success.

A successful recovery must never erase the fact that the primary operation failed.

## 18. Time and retry behavior

Test:

- immediate retry
- delayed retry
- repeated retry
- timeout
- dependency recovery
- transient verification failure
- permanent verification failure

Retries must not create duplicate mutations or amplify failure.

Every retry needs a reason.

Do not blindly retry destructive operations.

## 19. Boundary testing

Test:

- empty input
- one item
- maximum expected items
- malformed item
- missing item
- duplicate item
- unexpected extra item

For scripts operating on collections, explicitly test:

- zero resources
- one resource
- all resources
- partially matching resources

Never assume an empty result is safe.

For destructive operations, an empty selection must be treated as a potentially dangerous state and tested explicitly.

## 20. Destructive operations

Destructive automation requires a higher testing bar.

Test:

- correct target
- wrong target
- empty target
- stale target
- ambiguous target
- partially matching target
- target disappears
- dependency changes between selection and mutation

For destructive actions ask:

"Could a valid execution of this code destroy the wrong thing?"

Do not rely solely on unit tests.

Use realistic staging or fault-injection environments.

Name one destructive pattern specifically, because it is more dangerous
than an explicit delete command and easy to under-test: turn-down by
absence, where a resource is destroyed not because it was explicitly
targeted but because it is missing from a desired-state list (a
reconciliation loop that deletes "anything not in the current config").
Under this pattern, any failure that produces an incomplete or empty
desired-state list — a parser bug, a truncated read, a stale cache, an
authentication failure that silently returns zero entries — becomes an
implicit order to delete everything. Test this pattern with its own
scenarios: a parser that silently drops entries, a config source that
returns a truncated list, a source that fails and returns empty rather
than erroring, and a source that returns stale data. In every case verify
that the system distinguishes a confirmed, successfully-queried empty
state from a failed or incomplete one, and treats the latter as unsafe to
act on rather than as "nothing desired."

## 20a. Dry-Run Testing

Do not trust a `--dry-run` flag by name. Prove it does what it claims.

At minimum, test:

- **State comparison.** Capture the full relevant state before and after
  the dry-run invocation and assert it is byte-for-byte or semantically
  identical. A dry-run that prints "would delete 42 servers" but never
  demonstrates that zero servers were actually touched is an unverified
  claim, not a tested property.
- **Every layer, not just the entry point.** In a system with multiple
  layers (CLI, planner, cache, external API, mutator), a `--dry-run` flag
  handled only at the CLI layer does not prove the layers underneath are
  side-effect-free. Test each layer independently for mutation during a
  dry-run invocation — does the planner still write to a cache; does the
  call reach the external API at all; does the mutator receive and
  silently discard a request, or never get invoked?
- **Selection logic parity.** Confirm the dry-run evaluates the same
  target-selection logic the real run would use, not a simplified or
  stale approximation — a dry-run that reports on a different resource
  set than the real run would touch gives false confidence.
- **Read-only side effects.** A dry-run that only avoids mutating the
  primary target can still write local logs, temp files, or a cache; that
  may be acceptable, but it should be an explicit, tested, documented
  property rather than an accidental discovery.
- **Concurrent drift.** If state can change between a dry-run preview and
  the real run that follows it, test whether the tool re-validates at
  execution time or blindly trusts the preview.

## 21. Test the operator experience

Reliability includes the human using the tool.

Test whether an operator can determine:

- what is happening
- what changed
- whether it succeeded
- whether it degraded
- whether recovery ran
- whether recovery succeeded
- what action to take next

Test under time pressure where appropriate.

A tool that is technically correct but operationally confusing can still create incidents.

## 22. Work-as-done

Do not assume operators will use the tool exactly as designed.

Test realistic behaviors:

- retry after ambiguous output
- rerun after failure
- run recovery manually
- interrupt a command
- run commands concurrently
- use copied/generated artifacts later
- invoke the tool from an unexpected working directory
- operate with stale documentation

The test model must reflect real operational behavior, not only the designer's intended workflow.

## 23. Testing AI-assisted operations

If AI or agents participate in the operational workflow, test the system around the model rather than assuming deterministic model behavior.

Separate:

- probabilistic reasoning
- deterministic enforcement
- authorization
- mutation
- verification

The agent may propose.

Deterministic mechanisms must enforce what is allowed.

Tests should verify:

- unauthorized actions cannot execute
- malformed agent output cannot bypass validation
- prompt injection cannot expand authority
- tool boundaries are enforced
- actions are auditable
- human approval cannot be silently bypassed where required
- the agent cannot alter its own guardrails
- deterministic calculations remain deterministic
- the same incident may produce different reasoning paths without violating safety invariants

Do not test only whether the model gives a sensible answer.

Test whether the system remains safe when the model gives a terrible answer.

## 23a. Testing Interactions, Not Just Components

Every component passing its unit tests does not demonstrate the system is
safe. Design tests specifically aimed at interactions between components
that are each individually correct, not only at each component in
isolation. A validator, a deployment controller, and a rollback controller
can each pass every unit test written against their own contract while
combining, under a specific timing or state condition, into an unsafe
global outcome (see the STPA-style reasoning in sre-engineering-mindset
and sre-incident-review).

Concretely: run two or more individually-correct controllers together
under a scenario constructed to trigger their combined effect — not just
each one's fault injection separately. Examples: run an autoscaler and a
database-saturation condition together rather than testing the
autoscaler's scaling logic alone; run a rollback controller concurrently
with the deployment controller it might race against; run a validator
immediately followed by a target-identity change and confirm the
component that acts afterward re-checks rather than trusting the earlier
validation. A test suite that only covers components in isolation cannot
detect this failure class regardless of how thorough each component's own
coverage is. When blast radius is large enough that this class of failure
matters, testing also extends beyond the test suite into staged
production validation — see sre-release-deployment's canary and
progressive-delivery testing for that layer.

## 24. Invariants

Define properties that must remain true regardless of the agent's reasoning or execution path.

Examples:

- cannot modify resources outside declared scope
- cannot access secrets outside authorized scope
- cannot bypass authorization
- cannot exceed blast-radius limits
- cannot mutate without required approval
- cannot report success without verification
- cannot delete an ambiguous target
- cannot convert partial recovery into success

Test invariants under normal and adversarial conditions.

## 25. Regression discipline

Every discovered production or review defect should be evaluated for regression coverage.

Ask:

"Can this exact class of bug return without the test suite noticing?"

If yes, add a test.

Do not merely test the exact historical input if the underlying failure class is broader.

This applies equally to near misses — an automation that almost caused
serious damage but was stopped in time by an operator or by luck, with no
actual impact. A near miss reveals the same latent risk as a realized
incident; the only difference is that this particular run got caught
before the consequence landed. Treat it with the same regression
discipline as a defect that reached production: reproduce the condition,
add a test or invariant that would catch it automatically next time, and
do not wait for an actual outage to justify the coverage. Limiting
learning to incidents that caused real impact means learning only from
the cases where the system got unlucky, which is a much smaller and
less representative sample than the cases where a real weakness existed.

## 26. Test matrix

For important operations build a matrix covering:

| Dimension | Examples |
|---|---|
| State | initial, partial, degraded, failed |
| Dependency | healthy, unavailable, corrupted |
| Input | valid, empty, malformed, ambiguous |
| Execution | normal, retry, concurrent |
| Signals | none, SIGINT, SIGTERM, SIGKILL |
| Mutation | success, partial, corrupt |
| Verification | success, mismatch, unavailable |
| Recovery | success, partial, failed |
| Security | clean, secret present, injection |
| Operator | expected usage, retry, interruption |
| Artifact | valid, stale, missing, corrupted |

Do not attempt every Cartesian combination blindly.

Prioritize combinations based on blast radius, likelihood, and historical failure modes.

## 27. Test prioritization

Prioritize tests using:

Risk = Impact × Likelihood × Detectability gap

High-priority tests include:

- destructive mutations
- security controls
- recovery paths
- concurrency
- state transitions
- silent failure modes
- previously discovered bugs
- operations with large blast radius

Do not spend most of the test budget proving trivial happy paths.

## 28. Evidence standard

A test should produce evidence sufficient to support a claim.

Weak:

"Test passed."

Strong:

"Injected failure after resource B mutation; resource A remained correct, B was classified ATTEMPT_FAILED, C was evaluated, aggregate result became PARTIAL, exit code was 1, recovery artifact remained available."

Prefer evidence that is:

- reproducible
- observable
- attributable
- specific
- automation-friendly

## 29. Definition of done

A reliability-sensitive change is not complete merely because:

- code compiles
- lint passes
- happy path works
- unit tests pass

A stronger definition of done is:

- contract identified
- failure modes identified
- postconditions tested
- recovery tested
- exit status tested
- observability tested
- security implications tested
- concurrency considered
- interruption considered
- idempotency tested
- regression coverage added
- documentation matches behavior
- generated artifacts validated
- operational evidence captured

## 30. Final SRE testing question

Before declaring the system reliable, ask:

"If this operation fails at the worst reasonable moment, how will we know exactly what happened, how will the system avoid making the situation worse, and what evidence will prove that recovery actually occurred?"

If the answer depends on:

- guessing
- parsing vague logs
- trusting exit 0
- trusting a command's success
- assuming a trap always runs
- assuming operators behave ideally
- assuming dependencies remain available
- assuming generated code is correct
- assuming the AI is correct

the testing is not finished.