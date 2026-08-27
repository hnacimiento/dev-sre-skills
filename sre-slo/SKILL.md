---
name: sre-slo
description: >
  SLO and error-budget reasoning for Site Reliability Engineering: defining
  what "reliable enough" means for a system, measuring it honestly, and
  turning budget consumption into decisions that actually have teeth. Apply
  this skill whenever a system needs a reliability target, whenever toil or
  release velocity is being traded against reliability, whenever an incident
  or postmortem needs to translate into a policy change, or whenever an AI
  agent is asked to judge whether a system is "healthy enough" to keep
  shipping. This skill is the decision layer that sits on top of
  sre-observability (which supplies the raw signals) and feeds
  sre-release-deployment (which enforces the consequences) and
  sre-engineering-mindset (which supplies the toil/automation tradeoff this
  skill closes the loop on).
---

# SRE Service Level Objectives and Error Budgets

## 0. Purpose

This skill defines how to reason about *how much unreliability is
acceptable*, and what happens once that allowance runs out.

It is not a dashboard-configuration guide.

It is not a list of "good" SLO target numbers to copy.

It is not a monitoring skill — sre-observability already covers signal
collection, verification, and the danger of trusting agent-reported state.

It is the layer that turns those signals into a single question a team,
or an agent acting on a team's behalf, can actually answer under
pressure: *does this system have room left to take a risk, or not* — and
what is supposed to happen, mechanically, when the answer is no.

Read `sre-engineering-mindset` §1a before applying this skill's machinery
to anything. A one-off unattended installer script does not need a
service-level objective. A system with real, ongoing user traffic and a
release cadence usually does. This skill assumes the target has already
cleared that bar; it does not repeat the calibration argument, it only
depends on it.

---

## 1. What an SLO Is For

A Service Level Objective is not a description of how the system
currently performs. It is a decision instrument: a pre-committed
threshold that exists so that, in the moment reliability is trading off
against something else (a launch, a migration, a risky refactor, an
agent-proposed change), there is already an agreed answer instead of an
argument happening live, under pressure, with whoever is most senior or
most anxious in the room winning.

If an SLO never changes anyone's decision, it is decoration, not an
SLO. Before adopting one, be able to name at least one concrete decision
it is meant to gate ("if we breach this, we stop shipping features and
work only on reliability until we recover budget" is a decision; "if we
breach this, we'll discuss it" is not).

Corollary: an SLO with no attached consequence is not a lesser version of
a real SLO. It is a different artifact — a reporting metric — and should
not be described as an SLO, because doing so borrows credibility the
number has not earned.

---

## 2. Picking What to Measure (SLIs)

The Service Level Indicator is the actual measured signal the SLO is
built on. Getting this wrong invalidates everything downstream of it, so
it deserves more scrutiny than the target number itself usually gets.

Prefer indicators that:

- are measured as close to the user's actual experience as the
  architecture allows (a client-observed latency or success signal beats
  a server-side one, which beats a synthetic probe, in that order of
  preference, when all three are available);
- distinguish legitimate failure from disqualified traffic before
  counting it (see §10 on exclusions) rather than after — a metric that
  requires manual reinterpretation every time it is read is not a usable
  SLI;
- are unambiguous about what counts as success. "The request returned
  200" is not automatically success — see sre-observability §32,
  "Agent Output Is Not Evidence" and the broader point that an exit code
  or status code is a claim, not a verification. An SLI built on an
  unverified claim inherits that weakness silently.

A system usually has more than one thing worth an SLO (availability,
latency, correctness, freshness/staleness of data, durability). Resist
collapsing all of them into a single blended number — a blended
composite can stay green while one dimension that matters to a specific
user segment is failing, which defeats the purpose in §1: the whole
point was to have a number that reliably triggers the right decision.

---

## 3. The Error Budget: Formula and Rolling Windows

The error budget is the complement of the SLO target over a defined
window: if the objective is 99.9% success over 30 days, the budget is
the remaining 0.1% of that window's request volume (or time, for
availability-style SLOs) allowed to fail before the objective is
breached.

Two properties of the window matter more than the target number itself:

- **Rolling vs. calendar-fixed.** A rolling window (e.g., trailing 28
  days, recalculated daily) reflects current risk continuously. A
  calendar-fixed window (e.g., "this quarter") can create a perverse
  incentive near the boundary — burn the budget recklessly in the last
  days of a quarter because it resets tomorrow anyway. Prefer rolling
  windows unless there is a specific organizational reason (a fixed
  reporting cadence, a compliance requirement) to use a fixed one, and if
  a fixed window is used, say explicitly that the reset-day incentive
  exists and how it's being mitigated.
- **Window length vs. detection speed.** A long window (30–90 days)
  is stable and resistant to noise but slow to reflect a real ongoing
  regression — a genuine new failure mode can burn for days before the
  rolling average moves enough to matter. This is why burn-rate alerting
  (§4) exists as a separate mechanism layered on top of the raw budget,
  not a replacement for the window choice.

**Worked example.** A service with a 99.9% target over a 30-day rolling
window and 10,000,000 requests/month has a budget of 0.1% × 10,000,000 =
10,000 allowed failures for that window. If 25,000 failures have already
been recorded in the current window, the budget is not just consumed —
it is exceeded by 15,000 failures (the SLO has been breached for this
window, not merely "running low"). This is a bare arithmetic check that
should be verified by direct computation against real counts, not
estimated or asserted by an agent summarizing a dashboard (§12).

---

## 4. Multi-Window, Multi-Burn-Rate Alerting

A single "percent of budget consumed" number is a poor alerting signal on
its own: by the time a slow, sustained problem shows up clearly in a
30-day rolling window, a large fraction of a month's entire allowance may
already be gone, but a genuinely severe, fast-burning problem (all
traffic failing right now) needs to page someone in minutes, not surface
as a fractional percentage days later.

The standard resolution to that tension is alerting on the *rate* budget
is being consumed, evaluated over more than one time window at once,
rather than alerting only on cumulative percentage:

- A **short window** (e.g., 5 minutes to 1 hour) catches fast, severe
  burn — the kind of event that would exhaust a month's budget in hours
  if left unaddressed — and pages immediately.
- A **long window** (e.g., 6 hours to several days) catches slow,
  sustained burn — the kind that a short window's noise threshold would
  otherwise mask entirely — and can route to a less urgent channel
  (ticket, next-business-day) rather than a page.
- Requiring agreement between a short and a long window covering the
  same underlying condition (a common pattern: alert only when *both*
  a short-window burn rate and a longer-window burn rate exceed their
  respective thresholds) reduces false pages from brief blips that
  self-correct before they matter, without losing sensitivity to real
  incidents.

The exact multipliers and window pairs are a tuning exercise specific to
traffic volume and noise characteristics — do not present a single
"correct" set of numbers as universal. What is not optional is the
underlying shape: at least one fast/short pairing for severe burn and at
least one slow/long pairing for sustained burn, evaluated together
rather than as a single threshold.

---

## 5. Choosing the SLO Target: There Is No Universal Number

State plainly, every time this skill is applied: there is no target
percentage ("99.9%", "99.99%", "four nines") that is correct by default
for an arbitrary system. Presenting one as a starting point risks it
being adopted as a cargo-culted default rather than derived from
anything real. The target should come from:

- what users or downstream systems actually need and can detect —
  a target tighter than what any consumer can distinguish from a
  looser one is spending engineering effort on precision nobody uses;
- what the current architecture can plausibly sustain without
  extraordinary, unsustainable operational effort — an SLO the team can
  only meet by nonstop manual firefighting is not a target, it's a
  standing incident with a name;
- what a slightly-worse alternative would cost users or the business,
  stated as concretely as possible rather than assumed.

When none of that information is available yet — a new system, an early
project — say so explicitly rather than inventing a target. A defensible
starting move is to measure the SLI for a period without committing to a
target yet, observe actual achieved performance, and set an initial
objective slightly below current-observed performance (leaving
deliberate budget to spend), revisiting it on a stated schedule (§13)
rather than treating the first guess as permanent.

---

## 6. Freeze Policy: What "Consequences" Actually Means

An error budget policy is the pre-committed set of actions that happen
automatically (or are meant to) when budget is exhausted or burning too
fast. Per §1, this is the part that makes an SLO real rather than
decorative, and it is also the part most commonly skipped or left vague.

A freeze policy needs to specify, in advance and in writing, at minimum:

- what stops (feature releases, risky migrations, config changes of a
  certain blast-radius class — see sre-release-deployment's Destructive
  Changes section) and what is explicitly still allowed (security
  patches, active-incident fixes, the change that would restore budget);
- who has the authority to grant an exception, under what
  documented justification, and where that exception is recorded (an
  undocumented, verbal exception defeats the mechanism — it becomes
  whoever pushes hardest, which is exactly the pre-pressure problem §1
  exists to solve);
- what un-freezes it — a specific budget-recovery threshold or a
  time-boxed review, not an indefinite state that quietly becomes normal.

Cross-reference sre-release-deployment §28 for the release-engineering
side of this — the freeze is enforced there; this skill only defines
when it should trigger and why.

A policy that exists only as a sentence in a wiki page nobody reads at
the moment of pressure is functionally the same as having no policy.
State explicitly, when helping design one, whether it will actually be
checked automatically before a release proceeds (a real gate) or whether
it depends on someone remembering to look (a much weaker guarantee, and
one that should be named as such rather than assumed equivalent).

---

## 7. The Toil ↔ Error-Budget Bridge

`sre-engineering-mindset` §3 (Toil) already establishes that toil
displaces the engineering time that should go into things that reduce
future toil, and defers the deeper mechanics of the trigger to this
skill. Here is that trigger stated explicitly:

An exhausted error budget is one of the few triggers that should, by
policy, forcibly re-allocate engineering time away from feature work and
toward reliability work — including toil reduction — regardless of what
was previously planned. The budget is what gives that reallocation
legitimacy: without it, "stop and fix reliability" is a judgment call
someone has to win an argument to enforce; with it, it is the
pre-agreed consequence from §6 firing as designed.

This is also where the two concepts can be confused if not kept
distinct: toil (repetitive, automatable operational work) and error
budget (an allowance for imperfect reliability) are not the same axis.
A team can have severe toil with a healthy budget (things are working,
but a human is manually doing the same task every day to keep them
that way) or a healthy toil profile with an exhausted budget (things are
well-automated but currently failing anyway, for a genuinely new
reason). Do not conflate "we're burning budget" with "we have a toil
problem" without checking which one, or both, is actually true.

---

## 8. The 50% Rule Is an Alarm Threshold, Not a Sustained Target

Where a heuristic like "if toil exceeds roughly half of operational
capacity, that's a signal something structural needs to change" appears
in the broader corpus this skill draws on, treat it as a threshold that
should trigger investigation and course correction — not as an
acceptable steady state to plan around, and also not as a number to
defend precisely. The useful content of the rule is "there is a point
past which toil is crowding out the engineering work that would reduce
future toil, and that point is well below 100%, not just below it." The
specific fraction is a rule of thumb from one organization's context, not
a measured universal constant — restate it with that caveat rather than
as a hard fact (see §14 on epistemic humility, and
sre-engineering-mindset §38).

---

## 9. Error Budget Policy Needs Teeth Before It Needs a Dashboard

A common failure pattern: a team builds SLI collection, an SLO target, a
dashboard, and burn-rate alerts — real engineering effort — and stops
there, without ever writing the freeze policy in §6. The dashboard then
becomes a passive reliability report that is glanced at, not a decision
mechanism. This inverts the priority that actually matters: the policy
(what happens when budget is gone, and who enforces it) is the part
that makes the whole apparatus worth building. When resources are
limited, a crude SLI with a real, enforced consequence is worth more
than a precise SLI with none.

---

## 10. Legitimate Exclusions From Burn

Not everything that looks like a failed request should count against
the budget, but exclusions are also the most common place an SLO gets
quietly gamed into meaninglessness. An exclusion is legitimate only if it
is:

- defined *before* the incident it might apply to, not chosen
  retroactively once someone notices it would help the number (a
  postmortem is a reasonable place to *propose* a new exclusion rule for
  the future; it is not a place to reclassify what already happened);
- narrow and mechanically checkable (e.g., "requests during a
  scheduled, announced maintenance window with a specific tag" is
  checkable; "outages that weren't really our fault" is not);
- documented somewhere the same people who set the target can see it,
  so the achieved number and the target mean the same thing to everyone
  reading them.

A pattern worth naming explicitly because it recurs: excluding a
dependency's downtime from a service's own SLO on the theory that "it
wasn't our code" quietly turns the SLO from "what the user experienced"
back into "what our code did in isolation" — precisely the gap §2
warns against when it prefers user-observed signals. If a dependency's
failures are excluded, be explicit that the resulting number no longer
represents user-observed reliability, and consider whether the
dependency itself needs its own SLO with a real relationship (a
supplier agreement, an internal equivalent) to the consuming service's
budget.

---

## 11. SLOs for Small and Early Projects

Consistent with `sre-engineering-mindset` §1a: a formal multi-window
burn-rate alerting pipeline, a written freeze policy with a named
approval authority, and a quarterly review cadence are disproportionate
machinery for a project with one user, no on-call rotation, and no
release cadence to freeze. Do not present the full apparatus in this
skill as the minimum bar for "having an SLO."

What still transfers, even at small scale, and costs little to state:

- naming, even informally, what "acceptable" means for this system in
  concrete terms ("it's fine if this batch job is late by under an
  hour; it's not fine if it silently produces wrong output") — this is
  the core of §1 without any of the tooling;
- a stated, even if manual, answer to "what do we do if this keeps
  failing" — the floor version of §6, satisfied by a sentence rather
  than a documented policy with named approvers.

What does not transfer without real signal volume: burn-rate math
requires enough traffic for the rate to be statistically meaningful.
Applying multi-window alerting to a system that gets ten requests a day
produces noise, not signal — say so rather than recommending the
mechanism anyway because it is "best practice."

---

## 12. Agentic SRE and SLOs

Where an AI agent is involved in reasoning about error budgets or acting
on them, the same principle from sre-engineering-mindset and sre-security
applies: keep the deterministic and the probabilistic separated by
function.

- Computing whether budget is exhausted, and at what burn rate, is
  arithmetic over already-verified signals. That computation should be
  done deterministically (a real query against real metrics with a
  fixed formula), not delegated to an agent's judgment or summarization
  of a dashboard, per sre-observability §32 ("Agent Output Is Not
  Evidence") — an agent's *claim* that budget looks fine is not the same
  category of statement as a computed burn-rate number.
- Deciding whether to grant a freeze exception, whether a given failure
  qualifies for an exclusion under §10, or whether an SLO target should
  change, is a judgment call that can legitimately involve an agent's
  reasoning and recommendation — but per sre-security's Agent Threat
  Model, an agent recommending an exception is not the same as an agent
  having authority to grant one. If an agent is asked to evaluate "is
  the system healthy enough to keep shipping," it should compute the
  deterministic part itself or cite a computed source, state its
  confidence explicitly about the judgment part, and stop short of
  treating its own recommendation as the granted exception.

---

## 13. Reviewing and Revising SLOs

An SLO that is never revisited drifts out of relevance as the system,
its users, and its architecture change — it becomes as stale as any
other undermaintained artifact (cross-reference sre-observability §58 /
sre-incident-review on the Postmortem Nutrition Loop and agent playbook
staleness: an SLO target is exactly the kind of artifact that loses
accuracy silently if nothing forces a look).

Two triggers should force a review, not just a calendar:

- A scheduled cadence (quarterly is a common default, not a mandate —
  scale to how fast the system and its usage actually change);
- Any postmortem whose contributing factors include "the SLO didn't
  reflect what users actually needed" or "the exclusion rules masked
  the real signal" — a postmortem is a legitimate source of a *proposed
  future* target or exclusion change (see §10's distinction between
  proposing forward and reclassifying backward).

A target that is repeatedly missed is not automatically a target that
should be loosened — that direction of revision needs the same
justification in §5 as any other target choice (what changed about user
needs or sustainable architecture), not just "we kept missing it, so we
made it easier."

---

## 14. Common Anti-Patterns (SLO Theater)

Name these explicitly when reviewing an existing SLO setup, because each
one preserves the appearance of rigor while defeating its purpose:

- **Vanity target**: a number chosen because it sounds impressive
  ("five nines") rather than derived per §5.
- **Undead policy**: a freeze policy that exists in a document but has
  never actually blocked a release, with nobody able to say whether that
  is because budget has genuinely never been exhausted or because the
  policy isn't actually checked (§6, §9).
- **Silent exclusion creep**: exclusion rules added or widened after
  the fact to protect a specific incident's numbers (§10).
- **Blended-metric hiding**: a composite SLO that stays green while a
  real user-facing failure mode persists in one dimension (§2).
- **Frozen target**: an SLO set once at project inception and never
  revisited despite significant architecture or usage changes (§13).
- **Agent-asserted health**: treating an agent's summarized claim that
  "the system looks healthy" as equivalent to a computed budget state
  (§12).

---

## 15. Epistemic Humility When Applying This Skill

Distinguish, explicitly, between the parts of this skill that are
demonstrable technical mechanics and the parts that are organizational
or cultural judgment calls, per sre-engineering-mindset §38's extension
on stating confidence explicitly:

- The **arithmetic** of error budgets, rolling windows, and multi-window
  burn-rate detection (§3, §4) is a technical fact: given a window and a
  target, the budget and burn rate are computable and not a matter of
  opinion.
- **What target to pick, what to exclude, who gets exception
  authority, how often to review** (§5, §6, §10, §13) are organizational
  judgment calls informed by this corpus's conventions, not universal
  truths. When reasoning about them, say so — "a common convention is
  X, but this depends on your organization's risk tolerance and user
  base" — rather than presenting one convention as the only correct
  answer.
- The **50% toil-crowding heuristic** (§8) in particular should never be
  restated as a precise, load-bearing number without that caveat
  attached.

An agent or reviewer applying this skill should be willing to say
"I'm confident about the math here; I'm less confident this specific
target or policy is right for your context" in the same response,
rather than presenting both with equal certainty.

---

## 16. Final Questions / Definition of Done

Before treating an SLO setup as complete, it should be able to answer:

1. What SLI is measured, and does it reflect what users actually
   experience, or something easier to compute that stands in for it?
2. What is the target, and can its derivation (§5) be stated, or is it
   an unexamined default?
3. Is the budget tracked in a rolling window, and is burn-rate alerting
   (§4) in place with at least a fast/short and a slow/long pairing?
4. Does an exhausted budget trigger a specific, documented,
   actually-enforced consequence (§6), or only a conversation?
5. Are all exclusions (§10) pre-defined and checkable, or could any of
   them have been added after the fact to protect a number?
6. Is there a scheduled or postmortem-triggered review point (§13), or
   was this target set once and never revisited?
7. If an agent is involved in any of the above, is the deterministic
   arithmetic separated from the agent's judgment calls (§12), and is
   confidence stated explicitly wherever the answer is a judgment call
   rather than a computed fact (§15)?

A "no" to any of these is not necessarily disqualifying for a small or
early-stage system (§11) — but it should be a stated, deliberate
tradeoff, not a silent gap.
