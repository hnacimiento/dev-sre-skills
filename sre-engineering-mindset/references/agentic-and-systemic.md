# sre-engineering-mindset — Reference: Agentic and Systemic Reasoning (Sections 17–30)

This file is loaded on demand from `sre-engineering-mindset/SKILL.md` §1c,
when an AI agent will operate or trigger the automation, or when the
blast radius is high enough that component-level correctness cannot be
trusted to imply system-level safety. It is not a standalone skill — it
assumes the calibration in the parent `SKILL.md` has already run.

---

# 17. Agentic SRE / AI-Assisted Operations

Agents and LLMs are part of the automation layer.

They are not outside the reliability model.

Treat:

    human
    +
    automation
    +
    AI agent
    +
    infrastructure

as one sociotechnical control system.

Do not create an "AI exception" to normal SRE principles.

Apply the same questions:

- What can it observe?
- What can it infer?
- What can it change?
- What tools can it invoke?
- What authority does it have?
- What happens if its reasoning is wrong?
- What happens if its context is incomplete?
- What happens if its input is malicious?
- What happens if the model changes?
- What happens if the agent behaves differently on repeated runs?
- How do we know what it actually did?
- How do humans remain capable of operating without it?

---

# 18. Agent Authority

An agent may have operational autonomy without having organizational ownership.

Do not confuse:

    "the agent can execute X"

with:

    "the agent is responsible for deciding X should happen."

Separate:

    interpretation
    recommendation
    authorization
    execution
    verification
    accountability.

Where consequences are high, use explicit authorization boundaries.

Do not rely on a prompt such as:

    "never delete production"

as the primary safety control.

The restriction should be enforced by the surrounding system.

---

# 19. Agent Threat Model

An operational agent must be treated as a potential attack surface.

Threats include:

- prompt injection;
- malicious logs;
- malicious tickets;
- malicious configuration;
- poisoned documentation;
- untrusted external content;
- compromised tools;
- manipulated telemetry;
- credential exposure;
- excessive permissions;
- tool abuse;
- confused-deputy behavior;
- authorization bypass;
- accidental broad targeting;
- data exfiltration;
- policy bypass;
- context poisoning.

Never assume that data consumed by an agent is trustworthy merely because it
came from an internal system.

Separate:

    untrusted input

from:

    trusted control policy.

Enforce security at deterministic boundaries outside the model.

(Maintenance note: sre-release-deployment §43 and sre-security's Agent
Threat Model / Tool Boundaries sections carry adapted versions of this
same threat list. When adding a new threat category here, add it there
too rather than letting the lists silently diverge.)

---

# 20. Agent Evaluation

Do not assume that traditional deterministic unit tests are sufficient for
non-deterministic agents.

Evaluate agents using:

- representative incidents;
- historical incidents;
- postmortems;
- known failure cases;
- adversarial prompts;
- prompt-injection tests;
- tool misuse tests;
- regression datasets;
- repeated runs;
- scenario-based evaluation;
- human review;
- bounded experiments.

Measure not only:

    "did it eventually reach the correct answer?"

but also:

- Did it take an unsafe path?
- Did it expose secrets?
- Did it exceed authority?
- Did it make unnecessary mutations?
- Did it recognize uncertainty?
- Did it escalate appropriately?
- Did it preserve evidence?
- Did it recover correctly?
- Did it mislead the operator?

For agentic systems:

    correct outcome

is necessary but not sufficient.

The trajectory may matter.

---

# 21. Agent Drift

An agent can change behavior because of:

- model updates;
- prompt changes;
- tool changes;
- context changes;
- policy changes;
- knowledge changes;
- retrieval changes;
- environmental changes.

Therefore treat the agent as a changing production dependency.

Observe:

- behavior;
- tool invocation;
- failure modes;
- authority usage;
- latency;
- escalation rate;
- false positives;
- false negatives;
- unsafe recommendations;
- distribution shifts.

Do not assume yesterday's evaluation proves today's safety.

---

# 22. The Human Learning Loop

Automation should produce learning, not merely execution.

A healthy loop is:

    production
       ↓
    humans observe
       ↓
    incidents / decisions / surprises
       ↓
    postmortems
       ↓
    knowledge / tests / playbooks / policy
       ↓
    automation and agents improve
       ↓
    production

Postmortems are not merely organizational paperwork.

They are operational memory.

High-quality incident records can become valuable evaluation and regression
data for future automation and agents.

However, do not optimize postmortems merely for machine consumption.

The primary purpose remains human learning, organizational learning, and system
improvement.

---

# 23. Preserve Operational Exposure

Automate repetitive execution aggressively.

Do not automate humans out of understanding.

SREs need continuing contact with production reality through mechanisms such as:

- on-call;
- incident participation;
- debugging;
- operational reviews;
- GameDays;
- fault injection;
- system exploration;
- deployment observation;
- postmortem participation.

The goal is not:

    human executes everything.

The goal is:

    human understands enough to recover when automation is insufficient.

---

# 24. Observability Is Part of the Design

Do not add observability after implementation.

For consequential operations determine in advance:

- what happened;
- what changed;
- what was attempted;
- what succeeded;
- what failed;
- what remains uncertain;
- what the operator should inspect next.

A useful operational result should answer:

    WHAT happened?
    WHERE?
    WHEN?
    WHY?
    WHAT changed?
    WHAT is the current state?
    WHAT remains unsafe?
    WHAT should happen next?

Avoid requiring operators to reconstruct system state from arbitrary log lines.

---

# 25. Security and Reliability Are Coupled

Security failures can become reliability failures.

Reliability mechanisms can become security vulnerabilities.

Examples:

- rollback artifacts containing credentials;
- debug logs exposing secrets;
- overly broad recovery permissions;
- agents with excessive authority;
- unsafe automation endpoints;
- mutable shared state;
- insecure temporary files;
- unverified external inputs.

Evaluate both:

    "Can this fail?"

and:

    "Can someone cause this to fail?"

Do not assume a trusted environment.

Use specialized security practices when appropriate.

This mindset establishes the concern; the `sre-security` skill should provide
the deeper implementation guidance.

---

# 26. Concurrency and Shared State

If multiple actors can execute the operation concurrently, assume races exist
until proven otherwise.

Identify:

- shared files;
- shared resources;
- shared locks;
- caches;
- state databases;
- generated artifacts;
- external resources.

Ask:

    "What happens if two valid executions overlap?"

and:

    "What happens if one dies halfway through?"

Prefer kernel-enforced or transactional coordination over timing-based checks.

Do not mistake:

    "this normally happens sequentially"

for:

    "concurrency is impossible."

---

# 27. Failure Containment

A failure in one resource should not automatically prevent the system from
learning the state of every other resource.

Where safe, isolate failures by resource or operation.

Example:

    resource A -> restored
    resource B -> failed
    resource C -> restored

is more useful than:

    rollback failed

The operator needs to know the actual state.

However, containment must not create additional unsafe mutations.

The correct question is:

    "Can we continue safely?"

not:

    "Can we continue somehow?"

---

# 28. Recovery Must Be Observable

Never report recovery success based solely on intention.

A recovery result should distinguish at least:

    SUCCESS
    PARTIAL
    FAILED

and, where relevant before invocation:

    NOT_NEEDED

Once recovery has started, it must not silently return to a pre-invocation
sentinel.

Every resource that required recovery should have an explicit outcome such as:

    RESTORED
    REMOVED
    ATTEMPT_FAILED
    NOT_ATTEMPTED

The exact implementation may vary.

The principle does not.

---

# 29. STPA / System-Level Failure Thinking

Do not stop at:

    "each component is correct."

Ask:

    "What unsafe control action could emerge from their interaction?"

Reason about:

- controllers;
- controlled processes;
- feedback;
- sensors;
- operators;
- automation;
- timing;
- assumptions;
- authority;
- environmental conditions.

Look for failures where:

- a correct action is applied at the wrong time;
- a correct action is applied to the wrong target;
- a correct action is omitted;
- an action persists too long;
- feedback is delayed;
- feedback is stale;
- two correct components create an unsafe interaction.

This is especially important for:

- distributed systems;
- automation;
- deployment controllers;
- autonomous remediation;
- agents;
- orchestration;
- destructive operations.

---

# 30. Safety Boundaries vs System Complexity

Do not assume that adding more restrictions automatically increases safety.

A restriction can:

- prevent dangerous actions;
- force operators into unsafe workarounds;
- reduce required operational flexibility;
- hide important system state;
- create a new failure mode.

Likewise, flexibility is not automatically safe.

Evaluate both:

    What dangerous behavior does this boundary prevent?

and:

    What necessary behavior does this boundary prevent?

Design constraints intentionally.
