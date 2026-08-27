# sre-security — Reference: Fail-Open/Fail-Closed and Incident-Response Posture (sections 22-25a)

This file is loaded on demand from `sre-security/SKILL.md` §1a, whenever security and availability trade off against each other, including the revert-first vs. investigate-first hand-off and the three-rung access ladder. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-security's own core has already
been read.

---

## 22. Fail closed versus fail open

For each failure mode decide explicitly:

Should the system:

- stop?
- continue?
- retry?
- degrade?
- require human intervention?

Security-sensitive ambiguity should generally fail closed.

Examples:

- cannot verify target → stop
- cannot authenticate → stop
- cannot authorize → stop
- cannot verify integrity → stop
- cannot determine scope → stop
- cannot verify recovery → do not claim success

Do not blindly apply "fail closed" everywhere.

Availability-sensitive systems sometimes require controlled degraded behavior.

The correct behavior depends on the consequence of each failure.

---

## 23. Security versus availability

Security controls can themselves create availability risks.

Examples:

- expired credentials
- unreachable authorization service
- unavailable key service
- broken certificate validation
- overloaded security dependency

Analyze both directions:

"What happens if security controls fail?"

and:

"What happens if we bypass them?"

Never silently trade security for availability.

If a degraded mode exists, define it explicitly.

---

### 23a. Revert-First vs. Investigate-First: The Incident-Response Hand-off

SRE's default reflex during an outage is to restore availability
immediately — revert the last change, drain traffic, restart the failing
component — and investigate root cause afterward, from preserved
evidence. Security incident response has the opposite default: preserve
the compromised state, avoid tipping off an active adversary that they
have been detected, and investigate before any mutation destroys
forensic evidence. Applying the wrong default is costly in both
directions: reverting immediately during an active intrusion can alert
the attacker to move faster, destroy evidence of what they took or
changed, and push them toward something more destructive before they
lose access. Investigating cautiously during an ordinary software
failure needlessly extends a real outage waiting for a threat that is
not there.

This skill does not give you a deterministic switch for this — the
answer is a judgment call under uncertainty, and presenting it as a
clean algorithm would be a false guarantee. What it gives you is a set
of signals that should lower the threshold for suspicion and force an
explicit pause before defaulting to revert-first, and who that pause
transfers control to.

**Signals that should raise suspicion of an active adversary rather than
an ordinary failure** (no single signal is proof by itself; treat this
as evidence that shifts the prior, not a standalone trigger):

- unexplained authentication or authorization changes (new keys, new
  service accounts, altered ACLs) that no one on the team can
  immediately account for;
- access patterns from unexpected identities, locations, or times,
  especially involving elevated-privilege credentials;
- logs or audit trails that are missing, truncated, or altered in a way
  the failure mode itself does not explain;
- the failure's timing or shape correlates with a recent credential
  exposure, a known vulnerability disclosure, or an unexplained
  privilege escalation;
- behavior that looks deliberately evasive (throttled or staged actions,
  cleanup of traces) rather than the abrupt, undirected pattern typical
  of a bug or hardware fault.

**Hand-off criterion.** This is a judgment call, not a count to reach —
weigh the signals present rather than tallying them. A single signal
accompanied by evidence that data may have been exfiltrated, or that
persistence mechanisms (new accounts, new keys, new scheduled jobs) may
have been planted, already carries enough weight on its own to warrant
treating this as provisionally a security incident. Several of the
weaker signals occurring together, with no ordinary explanation any
responder can supply, can carry the same weight even without direct
evidence of exfiltration or persistence — the reasoning is "does the
totality of what we're seeing make an active adversary more likely than
an ordinary fault," not "did we clear a fixed number of boxes." When that
weighing crosses into genuine suspicion, treat this as provisionally a
security incident: stop any mutation beyond what is needed to contain
further damage, preserve the current state (no restart, no revert, no
deletion), and explicitly transfer incident command to a Security
Incident Commander before proceeding. This transfer is itself the safe
default under ambiguity — a brief, coordinated pause before a routine
revert costs far less than destroying evidence of, and alerting, an
active attacker during a real intrusion. State your confidence in this
judgment explicitly rather than presenting it with the certainty of a
threshold check — this is exactly the kind of contested, evidence-weighed
call sre-engineering-mindset §38 asks you to flag as a judgment call
rather than a demonstrable fact.

**Proportionality still applies.** Isolating or preserving a potentially
compromised resource is itself a mutation with a blast radius, and the
access tier used to perform it should scale with the three-rung ladder
in §25a rather than defaulting to full ZTP/MPA regardless of context. A
single low-blast-radius resource on a small project can be isolated
through rung 2 (brokered, audited access) with a clear record of who
acted and why; a mutation touching a shared, sensitive, or multi-tenant
boundary should require rung 3's typed proxy and multi-party
authorization specifically because the cost of getting containment wrong
at that scale is far higher. "This might be a security incident" changes
who decides and how cautiously — it does not exempt the decision from
the proportionality calibration in sre-engineering-mindset §1a.

Cross-reference: sre-incident-review §7a extends this into the
postmortem itself — a security-flavored incident changes what evidence
discipline requires during and after the response.

---

## 24. Human operator as part of the threat model

Humans are not automatically malicious.

They are nevertheless part of the system's failure surface.

Consider:

- fatigue
- urgency
- copy/paste errors
- ambiguous output
- confirmation fatigue
- misunderstanding
- stale documentation
- accidental privilege escalation
- unsafe retries

Design interfaces that make unsafe actions difficult.

Do not rely exclusively on:

"the operator should know better."

---

## 25. Zero-touch and privileged operations

Prefer controlled interfaces over unrestricted administrative access where practical.

A human should not need arbitrary production access merely because the automation lacks a better interface.

When privileged access is unavoidable, minimize:

- privilege
- duration
- scope
- available commands
- reachable resources

Audit the operation.

Do not treat SSH, sudo, root, or administrative APIs as inherently safe merely because the operator is trusted.

Evaluate the actual control boundary.

### 25a. The Access Ladder Has (at Least) Three Rungs, Not Two

Do not present this as a binary choice between raw interactive access and
full Zero Touch Production. In practice there is a middle rung most
organizations actually live on, and it is a legitimate destination, not
merely a waypoint.

**Rung 1 — Raw interactive access.** SSH/sudo/root with no broker in
front of it. A valid key or a trusted operator can run anything, and
there is little reliable record of what actually happened. Treat this as
the weakest acceptable state for anything touching production, and only
where blast radius is already small.

**Rung 2 — Brokered, audited interactive access.** SSH/sudo/root still
exist, but a session broker sits in front of them (examples: Teleport,
StrongDM, AWS SSM Session Manager, or an equivalent internal proxy). The
broker issues short-lived credentials instead of static keys, records the
full session, can require a ticket/justification before opening a
session, and can require an approver to unlock access to a sensitive
target. This does not stop an operator from typing an arbitrary
destructive command, but it gives you most of the auditability, fast
revocation, and blast-radius containment of full ZTP at a small fraction
of the engineering cost. For the large majority of teams and projects,
this is the correct steady state, not a stepping stone that must
eventually be abandoned.

**Rung 3 — Zero Touch Production / typed proxies.** Every operational
action is exposed as a narrow, typed API (`restart_service(id)` rather
than a free-form shell), with policy and multi-party authorization
enforced by the surrounding system rather than by operator discipline.
This buys the strongest guarantee — the model or operator cannot even
express an out-of-scope action — but it requires designing, building,
and indefinitely maintaining a purpose-built API for every operation
engineers need during an incident. That is a real, ongoing engineering
investment, not a one-time setup cost.

When reviewing a system, do not default to recommending rung 3 merely
because it is the strongest control. Ask whether the operations involved
are repetitive and standardized enough to justify building and
maintaining narrow APIs for them, and whether the organization has the
platform capacity to do so. A small or early-stage project that cannot
support that investment is not automatically insecure for stopping at
rung 2 — recommend rung 3 as an option to consider, explain what it would
cost to build, and let the actual owner of the project decide whether
that investment is warranted for their scale and risk profile. Do not
treat "still on rung 2" as an unresolved finding by default.

---

