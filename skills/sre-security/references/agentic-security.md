# sre-security — Reference: Agentic SRE Security (sections 26-34)

This file is loaded on demand from `sre-security/SKILL.md` §1a, whenever an AI agent participates in operations. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-security's own core has already
been read.

---

## 26. Auditability

Sensitive operations should leave sufficient evidence to answer:

- who initiated the action
- what was requested
- what authorization was used
- what resource was targeted
- what changed
- when it changed
- whether it succeeded
- whether recovery ran
- what recovery changed

Do not log secrets merely to make auditing easier.

Auditability and confidentiality must coexist.

---

## 27. Agentic SRE security

When an AI agent participates in operations, treat the agent as an untrusted decision-making component.

Do not grant the model authority merely because:

- it is highly capable
- it has high confidence
- it usually makes good decisions
- it follows the prompt
- it has passed benchmark tests

Separate:

agent reasoning

from:

authorization

from:

mutation

from:

verification

The agent may interpret and propose.

Deterministic controls should enforce what actions are permitted.

Name this failure class explicitly when reviewing agent architectures:
excessive agency — an agent holding more standing authority, more
reachable tools, or more autonomy to act without confirmation than the
task in front of it actually requires. Excessive agency is a design
property you can audit independently of whether any single action taken
so far has been unsafe; an agent can have excessive agency for months
before an input finally arrives that exploits it.

---

## 28. Prompt injection

Production data may contain attacker-controlled instructions.

Examples:

- logs
- tickets
- comments
- Git commits
- issue descriptions
- HTTP responses
- database content
- monitoring annotations

An agent reading these inputs must not automatically treat them as trusted instructions.

Test:

- direct prompt injection
- indirect prompt injection
- malicious logs
- malicious repository content
- tool output containing instructions
- conflicting instructions
- encoded instructions

The critical question is:

"Can data become authority?"

It must not.

---

## 29. Tool boundaries for agents

Agent tools should expose narrow capabilities.

Prefer:

restart_service(service_id)

over:

execute_shell(command)

Prefer:

get_deployment_status(deployment_id)

over:

ssh(host)

Prefer:

restore_resource(resource_id, version)

over:

run_as_root(command)

A tool should encode authorization and scope rather than delegating unrestricted power to the model.

(Related: sre-engineering-mindset §19 and sre-release-deployment §43/§44
carry their own Agent Threat Model / Agent Tool Boundaries sections built
on the same underlying principle, adapted to their own domain. Keep this
example set and those threat lists conceptually aligned when either is
revised.)

---

## 30. Agent authority escalation

Test whether an agent can:

- invoke unauthorized tools
- modify its own permissions
- alter policies
- modify guardrails
- retrieve secrets
- access unrelated resources
- invoke tools recursively
- chain individually safe actions into an unsafe result

Security boundaries must exist outside the model.

Never rely on the model to police its own authority.

---

## 31. Human-in-the-loop

Human approval is useful for high-impact operations, but it is not automatically a security boundary.

Test:

- approval bypass
- approval replay
- stale approval
- approval for wrong target
- approval after target changes
- confirmation fatigue
- ambiguous confirmation text

A human should approve a clearly identified action against a clearly identified target.

"Proceed?"

is weaker than:

"Delete production database X in environment Y?"

---

## 32. Agent determinism

Do not require an agent to produce identical reasoning for every equivalent situation.

Instead require deterministic enforcement of critical invariants.

Examples:

- maximum resource scope
- allowed API methods
- authorization
- secret access
- destructive action restrictions
- required approvals
- mandatory verification

Test multiple agent outputs against the same enforcement layer.

If different reasoning paths produce different unsafe authority, the architecture is too dependent on model behavior.

---

## 33. Deterministic exclusion principle

Do not use probabilistic reasoning where deterministic mechanisms are sufficient and stronger.

Prefer deterministic systems for:

- hashing
- parsing
- schema validation
- authorization
- arithmetic
- cryptographic verification
- resource selection
- policy enforcement
- integrity verification

Use probabilistic systems where they provide meaningful value:

- interpretation
- correlation
- summarization
- hypothesis generation
- natural-language reasoning
- adaptive investigation

The question is not:

"Can an LLM do this?"

The question is:

"Why would we accept probabilistic behavior here when deterministic behavior is sufficient?"

---

## 34. Security invariants

Define properties that must remain true regardless of implementation path.

Examples:

- unauthorized resources cannot be modified
- secrets cannot appear in public artifacts
- destructive operations cannot operate on ambiguous targets
- agents cannot bypass authorization
- recovery cannot deploy unverified artifacts
- generated scripts cannot execute outside declared scope
- a failed integrity check cannot be reported as success
- an unavailable authorization dependency cannot silently grant access

Test these invariants independently from individual components.

---

