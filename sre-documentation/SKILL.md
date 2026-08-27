---
name: sre-documentation
description: Apply an SRE documentation mindset to READMEs, architecture documents, runbooks, operational procedures, recovery guides, incident documentation, release notes, and public repositories. Treat documentation as an operational contract. Verify claims against implementation, tests, configuration, artifacts, and observed behavior. Document failure modes, recovery, blast radius, security boundaries, verification scope, limitations, ownership, automation, control loops, and agentic operations without overstating guarantees.
---

# SRE Documentation

## Purpose

Treat documentation as part of the operational system.

Documentation is not merely explanation.

It influences:

- how users operate the system
- what operators expect
- what engineers believe the system guarantees
- how failures are diagnosed
- how recovery is performed
- how changes are deployed
- how future maintainers modify the system
- how automation is trusted
- how security boundaries are understood

Before applying any of the structure below, size the documentation effort
to the same signals used to size engineering effort in
sre-engineering-mindset (section 1a): who runs this, what the worst
realistic outcome is, whether it is a one-off or reused infrastructure,
whether it touches production, credentials, or shared state. A twenty-line
script used by two engineers, with no production access, no credentials,
and no automation depending on it, needs a two- or three-line comment
explaining what it does and how to undo it — not a preconditions section,
a blast-radius analysis, a security-boundary writeup, and a runbook.
Applying the full structure in this skill to that script is not rigor, it
is bureaucracy that will be skipped or resented, which defeats the actual
goal of documentation being read and trusted. As the same signals move up
— shared use, production impact, credentials, agent involvement — apply
correspondingly more of the sections below. State briefly which level you
are applying and why, the same way sre-engineering-mindset asks for the
calibration to be stated rather than silently assumed.

A false documentation claim can become an operational failure.

The objective is therefore not to make the system sound safe.

The objective is to make the system's real behavior understandable, verifiable, and safe to operate.

---

# Core Principles

## 1. Documentation Must Match Reality

For every important claim ask:

    Is this actually demonstrated by the implementation?

Compare documentation against:

- source code
- tests
- configuration
- generated artifacts
- deployment behavior
- operational tooling
- logs
- observed runtime behavior

Never silently upgrade a claim beyond available evidence.

Examples requiring evidence:

    "rollback is automatic"

    "safe to run twice"

    "does not modify X"

    "credentials are never stored"

    "deployment is verified"

    "recovery is automatic"

    "operation is idempotent"

    "no downtime occurs"

A claim is not true merely because the implementation appears intended to provide it.

---

## 2. Separate Fact, Guarantee, Limitation, and Unknown

Distinguish explicitly between:

### FACT

What the implementation or observed system actually does.

### GUARANTEE

A property the system intentionally promises and can demonstrate within a defined scope.

### LIMITATION

Something the system does not protect against or cannot automatically verify.

### UNKNOWN

Something that cannot currently be determined from available evidence.

Never convert:

    UNKNOWN

into:

    SUCCESS

or:

    GUARANTEED

for the sake of cleaner documentation.

When evidence is insufficient, say:

    UNKNOWN

or:

    "This behavior is not verified automatically."

---

## 3. Claims Need Evidence

For important operational claims maintain the conceptual relationship:

    CLAIM
      ↓
    IMPLEMENTATION
      ↓
    TEST
      ↓
    EVIDENCE

Not every sentence requires a test.

High-risk claims do.

Examples:

    "safe to run twice"
        ↓
    idempotency implementation
        ↓
    repeated-execution test
        ↓
    observed state equivalence

    "rollback restores the previous state"
        ↓
    recovery implementation
        ↓
    rollback test
        ↓
    verified restored state

    "secret does not appear in artifacts"
        ↓
    artifact generation
        ↓
    artifact inspection test
        ↓
    verified absence

If a claim cannot be connected to evidence, narrow the claim.

## 3a. Weigh Competing Sources Before Writing a Claim

Documentation claims often get written by resolving a disagreement
between several sources that carry different strength as evidence, not by
consulting a single authority. In rough order of strength for
documentation purposes: a passing, currently-running test that exercises
the exact claim is the strongest evidence available. Direct inspection of
the current implementation is next — it shows what the code does today,
though not necessarily what it was intended to do or whether it is
covered by any test. An existing written claim (a prior README line, a
comment) is weaker still — it may simply be repeating an earlier,
unverified assumption. An operator's recollection ("I think it always
creates a backup") is the weakest source: valuable as a hypothesis to
check, not as something to transcribe into documentation as fact. When
these sources disagree, do not average them into a hedge or default to
whichever one happens to already be written down — identify which level
of evidence you actually have, and write the claim at that level (or mark
it UNKNOWN) rather than inheriting the confidence of the weakest source
that happens to have used the most confident wording. This mirrors the
evidence hierarchy in sre-incident-review; the same discipline about
not letting memory silently outrank direct inspection applies here.

---

## 3b. Where the Rest of This Reasoning Lives (Load-on-Demand Index)

Calibrate first with `sre-engineering-mindset` §1a and this skill's own
§0 sizing guidance above — blast radius, not document length, decides how
much of this skill's machinery is load-bearing. Everything past this
section now lives in five reference files under this skill's
`references/` folder, opened only when the calibration says they matter.

- **`references/audience-readme-and-scope.md`** (§4–§10) — documentation
  audiences, the README operational contract, documenting what changes,
  and documenting blast radius, failure states, recovery, and why exit
  codes are not state. Open this when writing or reviewing a README, or
  documenting scope and failure/recovery behavior.
- **`references/verification-state-and-architecture.md`** (§11–§20) —
  documenting verification scope, preconditions and postconditions,
  state models, operational invariants, architecture boundaries, control
  loops, why local correctness is not global safety, and automation
  documentation (including dry-run semantics and human interaction with
  automation). Open this when documenting verification, state,
  architecture, or automation.
- **`references/agentic-security-and-idempotency.md`** (§21–§29) —
  agents as part of the system, separating reasoning from authority,
  deterministic boundaries, agent security documentation, agent
  verification, security claims and evidence, credential/secret
  lifecycle, trust boundaries, and not casually claiming idempotency.
  Open this when documenting an AI agent's role, security claims, or
  idempotency.
- **`references/artifacts-runbooks-incident-and-drift.md`** (§30–§37) —
  documenting generated artifacts, known limitations as part of the
  contract, executable runbooks that preserve decision context, incident
  documentation (preserving operational reality, work-as-imagined vs.
  work-as-done), and detecting documentation drift. Open this when
  documenting artifacts, limitations, runbooks, or incident writeups, or
  auditing for drift.
- **`references/tradeoffs-review-learning-and-ownership.md`**
  (§38–§54) — documentation as a regression surface connected to tests,
  explaining tradeoffs and avoiding cargo-cult warnings, public
  repository trust, review procedure (including high-risk claims first),
  the publication checklist, the four operational questions, anti-pattern
  detection, change-impact participation, the human and AI learning
  loops, ownership, and the meta-rules for ambiguous documentation. Open
  this before publishing or considering documentation finished, or when
  deciding what an agent may learn from it.

Documentation whose subject scores low on `sre-engineering-mindset` §1a's
signals still owes itself that skill's three-item floor — it does not
need all five reference files read end to end.

---
