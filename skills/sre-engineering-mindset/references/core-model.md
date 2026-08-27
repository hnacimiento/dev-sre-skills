# sre-engineering-mindset — Reference: Core Model (Sections 2–16)

This file is loaded on demand from `sre-engineering-mindset/SKILL.md` §1c,
when §1a's calibration indicates the target is shared or production,
becomes reused infrastructure, or concurrent execution is possible. It is
not a standalone skill — it assumes the calibration in the parent
`SKILL.md` has already run.

---

# 2. Start With the Problem, Not the Requested Implementation

When someone says:

    "write a script"

do not immediately start writing the script.

First determine:

- What operational problem exists?
- Is this actually toil?
- How frequently does it occur?
- Who performs it?
- What decisions does the human currently make?
- Which parts are mechanical?
- Which parts require judgment?
- What is the intended final state?
- What happens if the operation is interrupted?
- What happens if the environment differs from expectations?
- Is the requested implementation the smallest reliable solution?
- Is there already a platform, API, reconciler, workflow, or mechanism that
  should be used instead?

Do not reject scripts merely because a declarative system would theoretically
be more elegant.

Do not force declarative architecture into situations where it increases
operational risk or slows a necessary mitigation.

Choose the simplest mechanism that provides the required reliability.

### Imperative vs Declarative: Explicit Decision Criteria

This choice is not aesthetic. Use these questions to decide, and revisit the
decision when the answers change:

- Does a reconciliation loop already exist for this resource (an operator,
  a controller, a platform API)? If yes, prefer expressing intent through it
  instead of a one-off imperative script that will drift from that loop's
  view of the world.
- Is this a repeated, steady-state operation or a single emergency mutation?
  Declarative/intent-based systems earn their cost through repetition and
  convergence. A one-time emergency migration under time pressure is often
  better served by a small, well-tested imperative script than by bending an
  unfamiliar declarative system to do something it was not built for.
- Does the declarative engine actually support the mutation you need? Forcing
  a declarative model to express an operation it cannot represent natively
  (a data migration, a one-off backfill) usually produces something less
  safe than a direct, reviewed imperative script.
- What happens if the imperative script and the declarative reconciler
  disagree about desired state afterward? If nothing reconciles that
  difference, you have created a hidden second source of truth.
- Can the imperative action be captured back into the declarative model once
  it is done, so the declarative source of truth remains accurate?

A useful default: prefer declarative/intent-based mechanisms for anything
that recurs or that other automation depends on; prefer imperative mechanisms
for genuinely one-off, reviewed, human-initiated actions, especially under
incident time pressure. Neither is universally safer — an imperative script
with no idempotency, no postcondition check, and no recovery plan is not
"simpler," it is just less examined.

---

# 3. Toil: Automate the Work, Not the Ritual

Toil is repetitive, manual, operational work that does not create durable value.

Automation should reduce toil, but removing human keystrokes is not the only
objective.

Ask:

    "What human work disappears after this automation exists?"

If the answer is:

    "the engineer still has to execute it, interpret the output,
     manually verify the result, and repeat the process"

then significant toil may still remain.

However:

    removing toil != removing humans from production

A mature SRE system should reduce unnecessary mechanical work while preserving
human understanding of production.

The objective is:

    fewer repetitive actions
    +
    better operational feedback
    +
    preserved situational awareness
    +
    greater system reliability.

Automation that makes engineers unfamiliar with the production system can
create a different failure mode: cognitive atrophy.

Toil reduction is not an end in itself. The organizational mechanism that
decides how much unreliability is acceptable, and what happens when that
tolerance is spent, is the error budget derived from an explicit SLO. If a
service's error budget is exhausted, that is a stronger and more concrete
signal to redirect engineering effort toward stability than any qualitative
sense that "there is a lot of toil right now." When reviewing or designing
automation, ask whether the team has an SLO with real consequences (a freeze
on risky changes, mandatory reprioritization) attached to its error budget.
Absence of that mechanism is itself a reliability gap, even if the toil
analysis in this section is otherwise sound. Deeper SLO and error-budget
mechanics belong in `sre-slo`; treat this as the connective tissue between
toil and that decision layer, not a replacement for it.

---

# 4. Preserve the Human Operational Model

Automation changes what humans do.

It does not remove humans from the sociotechnical system.

Whenever automation becomes more capable, ask:

- What knowledge will operators stop practicing?
- What signals will they stop seeing?
- What decisions will they stop making?
- What happens when automation fails?
- Will the human still understand the system well enough to recover?
- Does the interface help the operator understand what happened?
- Can an operator reconstruct the system's state after automation fails?

Avoid designing systems where:

    automation handles everything normally
    ->
    humans see almost nothing
    ->
    an unprecedented failure occurs
    ->
    humans suddenly regain control without context.

This is an out-of-the-loop failure mode.

Use mechanisms such as:

- meaningful telemetry;
- explainable operation summaries;
- dry-run or preview modes where useful;
- rehearsals;
- GameDays;
- controlled fault injection;
- incident simulations;
- operational exposure;
- postmortems;
- deliberate human review of important automated actions.

The purpose is not to maximize human intervention.

The purpose is to preserve human capability when it matters.

---

# 5. Work-as-Imagined vs Work-as-Done

Never assume that the workflow designed by engineers is identical to the workflow
used by operators.

Real operators work under:

- time pressure;
- incomplete information;
- conflicting priorities;
- alerts;
- outages;
- fatigue;
- organizational pressure;
- imperfect interfaces;
- undocumented constraints.

Therefore ask:

    "How will this actually be used during an incident?"

not only:

    "How was it designed to be used?"

A tool that looks safe during design review may become dangerous when an operator
is under pressure.

Test important operational interfaces with realistic users and realistic
conditions.

Observe how people actually use the system.

Do not punish operators for adapting a tool to reality.

Instead, treat deviations as evidence that the design or assumptions may be
wrong.

---

# 6. Clumsy Automation

Automation can reduce work during normal operation while increasing cognitive
load during incidents.

This is dangerous.

Evaluate automation across at least two operating modes:

    NORMAL
    INCIDENT

Ask separately:

### During normal operation

- Does it reduce toil?
- Does it reduce error?
- Does it improve consistency?
- Does it provide useful feedback?

### During failure

- Can the operator understand what happened?
- Can the operator determine what already changed?
- Can the operator stop it?
- Can the operator safely continue?
- Can the operator recover?
- Does the automation produce actionable diagnostics?
- Does it create additional coordination burden?

Never optimize automation solely for the happy path.

---

# 7. Intent and Desired State

Every meaningful automation should have an explicit notion of:

    CURRENT STATE
    DESIRED STATE
    TRANSITION
    VERIFICATION

Do not confuse:

    "the command executed"

with:

    "the desired state exists."

Prefer reasoning like:

    Given state X,
    perform controlled transition T,
    then verify that state Y actually exists.

This distinction is fundamental.

Exit status, HTTP 2xx, process completion, or successful API invocation are
evidence about an operation.

They are not automatically proof that the desired state exists.

---

# 8. Contracts and Postconditions

For every consequential operation define:

### Preconditions

What must be true before mutation?

### Mutation

What exactly changes?

### Postconditions

What must be true afterward?

### Failure state

What state may exist if the operation fails halfway through?

### Recovery

How can the system return to a known acceptable state?

### Observability

How can an operator determine what actually happened?

Prefer:

    "success = verified postcondition"

over:

    "success = command returned zero."

The system should not claim a state it cannot demonstrate.

---

# 9. Failure-First Thinking

Before implementing the happy path, ask:

    What if this fails here?

For each mutation point identify:

- what changed;
- what may have changed;
- what remains unchanged;
- whether the operation can safely continue;
- whether rollback is possible;
- whether rollback is itself fallible;
- how the resulting state is represented;
- what the operator must do next.

Think in state transitions, not merely sequential commands.

A useful abstraction is:

    INIT
      ↓
    VALIDATED
      ↓
    STAGED
      ↓
    MUTATING
      ↓
    VERIFYING
      ↓
    SUCCESS

with explicit failure transitions such as:

    FAILED_BEFORE_MUTATION

and:

    FAILED_AFTER_MUTATION
          ↓
       RECOVERY
          ↓
    RECOVERY_SUCCESS
    RECOVERY_PARTIAL
    RECOVERY_FAILED

Never hide the distinction between:

    nothing changed

and:

    something changed and recovery was attempted.

---

# 10. Partial Failure Is a First-Class State

Do not model operations as merely:

    SUCCESS / FAILURE

when partial state is possible.

Examples:

- 5 of 6 files updated;
- deployment reached 3 of 5 nodes;
- API mutation succeeded but verification failed;
- rollback restored some resources but not others;
- configuration changed but service restart failed;
- agent completed analysis but not execution.

Use explicit state representation.

A system must never report:

    SUCCESS

when the actual state is:

    PARTIAL

unless partial success is explicitly part of the contract and clearly
distinguished from complete success.

---

# 11. Recovery Is Its Own Operation

Rollback is not magic.

Rollback can fail.

Rollback can be partial.

Rollback can encounter a system state different from the one assumed by the
original operation.

Therefore:

    recovery itself requires
    preconditions
    mutations
    verification
    observability
    failure handling.

Recovery success must mean:

    "the recovery postconditions were verified."

Not:

    "the recovery commands returned zero."

If recovery cannot restore everything, say so explicitly.

Never allow:

    PARTIAL

to masquerade as:

    SUCCESS.

Never allow:

    NOT_ATTEMPTED

to masquerade as:

    NOT_NEEDED.

---

# 12. State Must Be Explicit

Whenever an operation can be interrupted, restarted, retried, or recovered,
define the relevant state explicitly.

Avoid relying on:

- log interpretation;
- timing;
- assumptions about what ran;
- process exit alone;
- stale cache files;
- implicit shell state.

Use explicit markers, manifests, state files, timestamps, resource identities,
or equivalent mechanisms where appropriate.

Distinguish:

    UNKNOWN
    NOT_STARTED
    STARTED
    COMPLETED
    VERIFIED
    FAILED
    PARTIAL
    RECOVERED

The exact vocabulary may vary.

The important property is that ambiguous states are not silently collapsed.

---

# 13. Idempotency

Ask:

    "What happens if this runs twice?"

and:

    "What happens if it runs again after failing halfway through?"

An operation should either be idempotent or explicitly define why repeated
execution is unsafe.

For destructive operations, ask a stronger question:

    "What happens if the selection is wrong, empty, stale, duplicated,
     broader than expected, or refers to a different environment?"

Idempotency is not sufficient by itself.

An idempotent destructive operation can still repeatedly perform the wrong action.

---

# 14. Blast Radius

Every automation should have a clearly understood blast radius.

Identify:

- resources affected;
- environments affected;
- users affected;
- data affected;
- credentials involved;
- dependencies touched;
- external APIs called;
- irreversible actions;
- shared state;
- concurrent operations.

Prefer bounded operations.

Prefer explicit targeting.

Prefer fail-closed behavior where ambiguity could cause destructive impact.

Never interpret:

    empty selection

as:

    everything

unless that behavior is explicitly and safely designed.

---

# 15. Turn-Up vs Turn-Down

Creation and destruction do not have symmetric risk.

Turn-up failures often create something that is unused.

Turn-down failures can destroy something that is actively serving production.

Therefore destructive operations deserve additional safeguards.

For deletion, decommissioning, replacement, migration, or cleanup:

- verify identity;
- verify scope;
- verify environment;
- verify selection;
- detect empty or unexpected selections;
- make destructive intent explicit;
- require stronger authorization where appropriate;
- provide preview/dry-run when useful;
- preserve recovery information;
- make irreversible actions difficult to trigger accidentally.

Do not apply identical safety assumptions to creation and destruction.

---

# 16. Deterministic vs Probabilistic Components

Use probabilistic systems where adaptation, interpretation, summarization, or
reasoning provides value.

Use deterministic mechanisms where the system needs enforceable guarantees.

Prefer deterministic mechanisms for:

- authentication;
- authorization;
- schema validation;
- hashing;
- exact comparisons;
- serialization;
- parsing structured data;
- resource limits;
- policy enforcement;
- transaction boundaries;
- safety interlocks;
- immutable audit records;
- exact calculations;
- invariant enforcement.

A useful principle:

    probabilistic for adaptation
    deterministic for enforcement.

However:

    deterministic component != deterministic system.

A deterministic validator can be perfectly correct while the overall control
system still becomes unsafe because of incorrect assumptions, stale context,
unexpected interactions, or emergent behavior.

Therefore use both:

    local deterministic guarantees
    +
    system-level reasoning about interactions.
