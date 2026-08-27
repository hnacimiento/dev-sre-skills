# sre-incident-review — Reference: Agentic SRE, Observability, Recovery, Severity, and Counterfactuals (sections 18-29)

This file is loaded on demand from `sre-incident-review/SKILL.md`'s
load-on-demand index, whenever an agent participated in the incident or its response, or detection/recovery/severity need analysis. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-incident-review's own core has already been read.

---

# Agentic SRE

## 18. Agents are part of the system

When an AI agent participates in operations, treat it as:

- a system component
- a control-system participant
- a sociotechnical actor
- a security boundary
- part of the threat model

Analyze:

- inputs
- context
- available context
- missing context
- tools
- permissions
- policies
- decisions
- recommendations
- actions
- outputs
- feedback
- limits
- supervision
- uncertainty handling
- failure behavior

Do not treat the agent as a magical or autonomous black box.

---

## 19. Separate reasoning from authority

Distinguish:

OBSERVATION

- what data the agent received

INTERPRETATION

- what the agent believed was happening

RECOMMENDATION

- what the agent proposed

AUTHORIZATION

- what human or policy mechanism authorized the action

EXECUTION

- what mutation actually occurred

VERIFICATION

- what evidence demonstrated the resulting state

An agent may have significant reasoning autonomy without having unlimited operational authority.

Never assume:

    recommendation = authorization

or:

    execution = success

or:

    exit 0 = desired state

---

## 20. Deterministic boundaries

Do not use probabilistic reasoning where deterministic mechanisms are required for guarantees and a suitable deterministic alternative exists.

Prefer deterministic mechanisms for:

- authorization
- schema validation
- access control
- allowlists
- hashing
- critical calculations
- structured parsing
- state comparison
- resource limits
- blast-radius limits
- invariant enforcement
- policy enforcement
- execution boundaries

Use probabilistic reasoning where it provides useful adaptation, such as:

- interpretation
- synthesis
- correlation
- hypothesis exploration
- summarization
- option generation

Remember:

    deterministic component != deterministic system safety

Local deterministic guarantees provide bounded controls.

They do not eliminate system-level emergent behavior.

---

## 21. Agent incident analysis

If an agent participated in an incident, ask:

- What context did it receive?
- What context was missing?
- What tools were available?
- What tools were unavailable?
- What permissions did it have?
- What permissions should it have had?
- What did it observe?
- What evidence did it use?
- What evidence did it ignore?
- What did it recommend?
- What did it actually execute?
- Was there prompt injection?
- Was there misleading or adversarial information?
- Was there automation bias?
- Was there relevant nondeterministic behavior?
- Which guardrails worked?
- Which guardrails failed?
- Which policy should have stopped the action?
- Which deterministic boundary should have contained the action?
- What feedback did the agent receive?
- What feedback did it fail to receive?

Do not automatically attribute the incident to the model.

Analyze the complete system around the agent.

---

## 22. Agent security is part of incident analysis

When an agent has production access, consider:

- prompt injection
- malicious context
- compromised tools
- excessive permissions
- confused-deputy behavior
- indirect instruction attacks
- credential exposure
- tool abuse
- unauthorized action chaining
- policy bypass
- data exfiltration
- manipulation of observability
- manipulation of agent context

The agent must be treated as an operational capability that can become an attack surface.

Do not assume human approval alone eliminates these risks.

Analyze whether the approval mechanism itself was presented with trustworthy evidence.

---

# Observability

## 23. Detection is part of reliability

Do not analyze only:

    "What caused the incident?"

Also ask:

    "Why did it take X minutes to know that the problem existed?"

Analyze:

- detection latency
- alert quality
- signal coverage
- false positives
- false negatives
- observability gaps
- diagnostic latency
- recovery signals
- operator visibility
- signal ambiguity
- telemetry correctness

---

## 24. Detection vs diagnosis vs recovery confirmation

Distinguish:

DETECTION

- evidence that a problem exists

DIAGNOSIS

- understanding what is happening

RECOVERY CONFIRMATION

- evidence that the intended problem has actually been resolved

A system can have excellent detection and poor diagnosis.

A rollback can complete successfully without demonstrating recovery.

Never confuse:

    "the command completed"

with:

    "the system reached the desired state."

---

# Recovery Analysis

## 25. Recovery is a first-class system property

Analyze recovery with the same rigor as prevention.

Ask:

- Did rollback exist?
- Was rollback actually reversible?
- Was prior state available?
- Were backups available?
- Were backups verified?
- What happened if rollback partially failed?
- How was partial failure detected?
- What state remained afterward?
- Could the system be recovered manually?
- Could the system enter an UNKNOWN state?
- What happened under SIGKILL?
- What happened under process termination?
- What happened under network loss?
- What happened under storage failure?
- What happened when dependencies were unavailable?

Recovery should be considered part of the design, not an afterthought.

---

## 26. Recovery success must be demonstrated

Never treat:

    command exit 0

as automatically equivalent to:

    system recovered

Recovery must be supported by observable postconditions.

Examples of recovery evidence include:

- state comparison
- health verification
- read-after-write verification
- API confirmation
- checksum comparison
- replica convergence
- application-level validation
- user-visible success signals

If recovery is partial:

    PARTIAL

If recovery cannot be established:

    UNKNOWN

Never convert UNKNOWN into SUCCESS for narrative convenience.

---

# Severity and Impact

## 27. Impact before blame

First characterize:

- users affected
- duration
- percentage affected
- regions affected
- services affected
- data affected
- availability loss
- integrity loss
- confidentiality impact
- blast radius
- customer-visible symptoms

Separate:

IMPACT

from:

CAUSE

and:

CONTRIBUTORS

Do not use severity as a proxy for culpability.

---

## 28. Blast radius

Determine:

- actual blast radius
- maximum possible blast radius
- what limited expansion
- what enabled expansion
- which controls could have reduced impact
- which actions increased impact
- which boundaries were absent
- which boundaries were bypassed

Pay particular attention to destructive operations.

For destructive actions, analyze:

- target selection
- empty-set behavior
- default behavior
- authorization
- scope validation
- dry-run behavior
- blast-radius limits
- reversibility
- confirmation
- post-action verification

---

# Counterfactual Analysis

## 29. Use counterfactuals carefully

Counterfactuals can reveal missing controls.

Examples:

    "What would have happened if the deployment affected only 1%?"

    "What would have happened if the health check detected this signal?"

    "What would have happened if rollback partially failed?"

    "What would have happened if the operator had not intervened?"

    "What would have happened if the agent had recommended the opposite action?"

Counterfactuals are hypotheses, not facts.

Clearly label them as hypothetical scenarios.

Use them to identify:

- missing controls
- unsafe assumptions
- blast-radius opportunities
- recovery weaknesses
- alternate failure paths

---

