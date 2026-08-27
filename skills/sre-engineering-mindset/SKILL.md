---
name: sre-engineering-mindset
description: >
  Core Site Reliability Engineering reasoning framework for designing, reviewing,
  modifying, operating, automating, documenting, testing, deploying, and evolving
  production systems. Apply this mindset whenever reliability, operational risk,
  automation, failure handling, production behavior, human factors, or system
  interactions matter. This skill is intentionally technology-agnostic and acts
  as the parent mental model for specialized SRE skills such as bash, testing,
  security, observability, deployment, documentation, and incident review.
---

# SRE Engineering Mindset

## 0. Purpose

This skill defines how to THINK like an SRE before deciding how to implement
something.

It is not a checklist of commands.

It is not a coding style guide.

It is not a collection of Bash best practices.

It is not a replacement for specialized engineering skills.

It is the reasoning layer that determines:

- what problem is actually being solved;
- whether automation is appropriate;
- what the system's desired state is;
- what can go wrong;
- what the blast radius is;
- what must remain deterministic;
- what must be observable;
- what recovery means;
- what humans must still understand;
- how the system behaves when assumptions become false;
- how the system learns from real production experience.

The goal is not:

    "make the operation work."

The goal is:

    "make the system capable of performing the intended operation,
     detecting when reality differs from intent,
     containing failure,
     recovering where possible,
     communicating its actual state,
     and remaining operable by humans when assumptions fail."

---

# 1. Core Mental Model

Treat every production automation, script, tool, service, agent, deployment
mechanism, or operational workflow as part of a larger sociotechnical system.

The real system is:

    humans
      ↕
    automation
      ↕
    software / agents
      ↕
    infrastructure
      ↕
    production state
      ↕
    users / external systems
      ↕
    adversaries / unexpected conditions

Reliability does not live exclusively inside any one component.

A component can behave exactly according to specification while the overall
system still enters an unsafe or unavailable state.

Therefore:

    component correctness != system reliability

and:

    deterministic behavior of a component
    !=
    deterministic behavior of the whole system.

Think in terms of control loops, feedback, state transitions, interactions,
constraints, and recovery.

## 1a. Calibrate the Situation Before Applying the Rest of This Mindset

Run this calibration step before invoking the deeper machinery in this
skill or in any specialized skill it coordinates (sre-bash, sre-testing,
sre-security, sre-observability, sre-release-deployment). The size of the
codebase and the number of skills loaded into context are not the
throttle. Blast radius is the throttle.

Before going further, size the situation using concrete signals:

- **Audience.** Is a human present and watching in real time, or is this
  unattended/scheduled where nobody will notice an obviously wrong result
  until later?
- **Blast radius.** What is the worst realistic outcome — one local file
  on one machine, or something shared, production, or belonging to
  someone else?
- **Lifetime and reuse.** Is this a one-off that gets run once and
  discarded, or will it become infrastructure other people or other
  automation depend on?
- **Reversibility.** Can a bad run simply be re-run or undone trivially,
  or does it destroy something that cannot be easily recreated?
- **Concurrency exposure.** Could this realistically run twice at once,
  by design or by accident?
- **Trust boundary.** Does it touch secrets, untrusted network content,
  or a system outside your own control?

A situation that scores low across all of these — an attended installer,
run by one operator, on one machine, trivially re-runnable, touching no
secrets and no shared state — genuinely does not need a formal state
machine, distributed locking, generated rollback automation, an agent
threat model, or an exhaustive fault-injection matrix. Reaching for that
machinery anyway is the same mistake as cargo-culting Zero Touch
Production onto a two-person project: it does not make the script safer,
it makes it slower to write and harder to read, for a risk that was never
present.

Regardless of how low the situation scores, a small floor still applies
to every script, because it is cheap and prevents most real incidents on
its own: the script must not report success when it did not verify its
result (no `echo "done"` masking a swallowed failure); it must not treat
an empty or ambiguous target as "everything" for a destructive operation;
and it must tell the operator what happened when something goes wrong,
well enough to act on it. These three are not "the low end of the scale"
— they are the baseline every script owes its user, from a five-line
installer to a production deployment controller.

As the signals move up — the job becomes unattended, the target becomes
shared or production, other automation starts depending on it, secrets or
untrusted input enter the picture — name explicitly which additional
machinery actually becomes load-bearing, rather than applying all of it
by default:

| Signal that increases | What becomes worth doing |
|---|---|
| Unattended / scheduled | Truthful exit codes and operator-actionable failure output become mandatory, since no human is watching live to catch a silent wrong result (see sre-observability). |
| Shared or production target | Blast-radius analysis, explicit preconditions, and destructive-operation safeguards become worth the effort (see `references/core-model.md` §14-15, sre-security). |
| Becomes reused infrastructure | Idempotency, explicit state, and a real recovery contract earn their cost (`references/core-model.md` §7-13, sre-bash). |
| Concurrent execution possible | Locking and shared-state analysis become necessary, not optional (sre-bash concurrency sections). |
| Secrets or untrusted input involved | Threat modeling and the security sections become load-bearing regardless of how small the script otherwise is (sre-security). |
| Will be operated or triggered by an AI agent | Agent authority, tool boundaries, and drift/evaluation sections apply regardless of scale (`references/agentic-and-systemic.md` §17-22, sre-security). |

State the calibration explicitly and briefly before diving into
implementation or review — "this is an attended, low-blast-radius,
single-machine installer; applying the floor only" is itself useful
output, not throat-clearing. It lets the person you are working with
correct your sizing if they know something about the context that you do
not, instead of silently discovering thirty sections of unrequested
machinery in the response, or silently discovering that a genuinely
risky script was reviewed as if it were a toy.

---

## 1b. Six Questions for Consequential Automation (Cognitive Router)

This section names six questions a reasoner should hold before
consequential automation, and points to exactly where each one's
mechanics live across this skill set. It is a router, not a duplicate —
follow the reference, do not stop at the one-line summary here.

These are six lenses to reason through, not six boxes to tick in
sequence. They are listed in one order for readability, but they are not
a pipeline: real reasoning moves back and forth between them as
understanding changes, the same way §40's SRE Decision Loop is a loop
and not a line. A system that "passed" all six has not thereby been
proven safe — that illusion is exactly what STPA below exists to guard
against, and it is the same trap named explicitly in
sre-release-deployment's "The Checklist Trap" section. Six questions
satisfied on paper is not the goal; six questions genuinely reasoned
through, and revisited when something surprises you, is.

Depth is gated by §1a's blast-radius calibration above, not by the
length of this list. Applied to a 40-line unattended install script
touching one local file, these six questions are a few sentences of
reasoning, most of them trivial. Applied to a multi-tenant,
revenue-bearing deployment, the same six questions are a real design
document. Do not apply uniform depth regardless of what §1a's signals
say about the situation in front of you.

**Does this reduce toil, or just relocate it?** (`references/core-model.md`
§3 Toil). Before designing anything: does this eliminate real, recurring
work, or does it relocate the same manual work behind a script someone
will still have to run, watch, and occasionally fix by hand?

**What could the interaction between correct components still get wrong?**
(`references/agentic-and-systemic.md` §29 STPA; sre-incident-review
§11–§12 for the equivalent reconstruction done after the fact). Do not
evaluate a component in isolation. Ask how it can interact with the rest
of the system to produce a loss even when every individual component is
functioning exactly as specified. For low blast radius, this can be one
sentence ("no interaction surface beyond this local file"). For high
blast radius, this needs the same rigor sre-incident-review §11–§12
applies retroactively during a postmortem — done here, before the fact,
instead.

**Does the access tier match the direction and size of the blast radius?**
(`references/core-model.md` §14 Blast Radius and §15 Turn-Up vs
Turn-Down; sre-security §25a for the three-rung access ladder). Turn-up
and turn-down are not symmetric, and constructive and destructive
operations should not default to the same access tier. Pick the rung
proportional to blast radius per sre-security §25a — not by rote
escalation to the strongest control available.

**What counts as success, failure, and partial failure, before the
mutation runs?** (`references/core-model.md` §8 Contracts and
Postconditions, §9 Failure-First Thinking, §10 Partial Failure). Define
these before writing the mutation. An exit code is a claim, not proof
that the desired state was reached — see sre-observability's
verification content for the difference.

**Where does this need a computed guarantee instead of a judgment call?**
(`references/core-model.md` §16 Deterministic vs Probabilistic
Components; sre-security §33 Deterministic Exclusion Principle;
sre-slo §12 for how this applies specifically to burn-rate judgment).
Probabilistic reasoning — human judgment, an LLM agent's interpretation
— is the right tool for adapting to ambiguity. Deterministic mechanisms
are required wherever the system needs a verifiable guarantee: a
calculation, a hash, an authorization check, the mutation itself. An
agent's confidence is never a substitute for a computed guarantee.

**What will this leave a human, or an agent, less able to understand or
recover, and does that get fed back?** (§4 Preserve the Human
Operational Model in `references/core-model.md`, §22 Human Learning Loop
in `references/agentic-and-systemic.md`; sre-incident-review,
sre-observability §58, and sre-release-deployment §57 for the Postmortem
Nutrition Loop these three skills keep synchronized). Automation should
not remove the human from the cognitive loop entirely. Every
consequential incident should feed a blameless postmortem that keeps
both the operator's skill and any participating agent's evaluation data
current.

---

## 1c. Where the Rest of This Reasoning Lives (Load-on-Demand Index)

Everything that used to follow §1b inline in this single file now lives
in three reference files under this skill's `references/` folder, read
only when §1a's calibration says they are load-bearing. This keeps the
part of this skill that is always loaded small enough that reaching the
calibrator itself does not require reading forty-plus sections first —
the same proportionality this skill asks of every automation applies to
the skill's own packaging.

**If §1a's floor is all that applies** — attended, low blast radius,
single machine, trivially re-runnable, no secrets, no shared state, no
agent involved — stop here. Apply the three-item floor from §1a and
reason through the six questions in §1b at the "one sentence per step"
depth. There is no need to open any reference file below for a
situation at this level.

**If any signal in §1a's table is elevated, open the matching file(s):**

- **`references/core-model.md`** — sections 2 through 16: starting with
  the problem, toil, preserving the human operational model,
  work-as-imagined vs. work-as-done, clumsy automation, intent and
  desired state, contracts and postconditions, failure-first thinking,
  partial failure as a first-class state, recovery as its own operation,
  explicit state, idempotency, blast radius, turn-up vs. turn-down, and
  the deterministic-vs-probabilistic (Hybridization) rule. Open this
  whenever the target is shared or production, becomes reused
  infrastructure, or concurrent execution is possible.
- **`references/agentic-and-systemic.md`** — sections 17 through 30: the
  full agent authority, threat model, evaluation, and drift treatment,
  the human learning loop, preserving operational exposure,
  observability as design, the coupling of security and reliability,
  concurrency and shared state, failure containment, recovery
  observability, STPA / system-level failure thinking, and safety
  boundaries vs. system complexity. Open this whenever an AI agent will
  operate or trigger the automation, or when the blast radius is high
  enough that component-level correctness cannot be trusted to imply
  system-level safety.
- **`references/execution-and-closing.md`** — sections 31 through 43:
  dry-run and preview, testing the system rather than just the code,
  documentation as an operational interface, releases and deployment,
  incident thinking, ownership, the do-not-overfit-to-Google and
  avoid-absolute-rules cautions (including the explicit
  confidence-calibration instruction), resolving conflicts between
  principles, the SRE decision loop, the Final SRE Questions, what this
  skill does not do, and the definition of done. Open this before
  calling any consequential automation finished, once the situation
  scored above the floor in §1a.

For anything already scoped by §1b's cross-references directly to a
sibling skill's named section (sre-security §25a or §33, sre-slo §12,
sre-incident-review, sre-observability, sre-release-deployment), go
straight there — there is no need to open this skill's own reference
files first if the question is already answered by a sibling skill's
section.

---
