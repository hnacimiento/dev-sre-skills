# sre-testing — Reference: Operator Experience, Agentic Testing, and Final Review (sections 21-30)

This file is loaded on demand from `sre-testing/SKILL.md`'s load-on-demand
index, when testing the operator experience, AI-assisted operations and their interactions, or before considering a test suite finished. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-testing's own core has already been read.

---

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