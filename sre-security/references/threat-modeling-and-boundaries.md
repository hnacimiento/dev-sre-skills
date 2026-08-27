# sre-security — Reference: Threat Modeling and Trust Boundaries (sections 2-8)

This file is loaded on demand from `sre-security/SKILL.md` §1a, whenever a threat model, trust boundary, or blast-radius analysis is needed before implementation. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-security's own core has already
been read.

---

## 2. Threat model before implementation detail

Before reviewing individual commands, identify:

- assets
- actors
- trust boundaries
- entry points
- privileged operations
- sensitive data
- dependencies
- external systems
- recovery paths
- generated artifacts
- operators
- automation
- agents
- adversarial inputs

Ask:

"What can influence this system?"

Then ask:

"What can that influence cause?"

Do not assume that only explicit user input is untrusted.

Potentially hostile or unreliable inputs include:

- API responses
- logs
- configuration files
- downloaded artifacts
- environment variables
- filenames
- container metadata
- generated files
- repository contents
- external dependencies
- cached state
- stale state
- model output
- tool output
- telemetry
- operator-provided arguments

---

## 3. Assets and consequences

Identify what must be protected.

Examples:

- credentials
- API keys
- tokens
- certificates
- private configuration
- production data
- infrastructure state
- deployment artifacts
- recovery artifacts
- audit logs
- operator identity
- authorization state

For each asset ask:

- Can it be read?
- Can it be modified?
- Can it be deleted?
- Can it be replayed?
- Can it be exposed through logs?
- Can it be exposed through process arguments?
- Can it appear in generated files?
- Can it persist after an operation?
- Can an attacker use it for lateral movement?
- Can recovery accidentally expose it?

Security analysis must consider consequences, not only the presence of a vulnerability.

---

## 4. Trust boundaries

Explicitly identify transitions between trust domains.

Examples:

operator
→ CLI

CLI
→ script

script
→ host

host
→ container

container
→ API

repository
→ build system

LLM
→ tool

agent
→ production

production
→ recovery environment

At every boundary ask:

- What crosses the boundary?
- Who controls it?
- Is it authenticated?
- Is it authorized?
- Is it validated?
- Is it escaped?
- Is it logged?
- Can it be replayed?
- Can it contain attacker-controlled content?

Never assume that because two components belong to the same organization they belong to the same trust domain.

---

## 5. Least privilege

Every actor and automation component should have only the authority required for its operation.

Review:

- filesystem permissions
- API permissions
- container permissions
- cloud IAM
- service accounts
- sudo privileges
- tokens
- SSH access
- administrative APIs
- agent tools

Ask:

"If this credential or process is compromised, what is the maximum damage it can cause?"

That answer defines part of the effective blast radius.

Do not stop at:

"The operation currently needs root."

Ask:

"Does the entire process really need root?"

And:

"Can the privileged operation be isolated behind a smaller interface?"

Prefer narrowly scoped capabilities over unrestricted administrative access.

Avoiding the "root is always forbidden" dogma requires naming when broad
privilege can still be acceptable, not just discouraging it in the
abstract. Treat root/broad privilege as acceptable when most of the
following hold together, not any single one in isolation: the operation's
actual action surface genuinely spans most of what root grants (so
isolating a narrower interface would not meaningfully shrink blast
radius); the blast radius is already bounded by other means (single host,
no network exposure, no shared credentials with other systems); the
elevated session is short-lived, freshly authenticated, and audited
rather than a standing grant; and there is a concrete, budgeted plan to
narrow the interface later if the operation becomes routine rather than
exceptional. Where none of these hold, "we run it as root for
simplicity" is a cost being paid by the blast radius, not a neutral
implementation shortcut.

---

## 6. Authorization versus authentication

Do not confuse:

"Who are you?"

with:

"What are you allowed to do?"

A valid credential must not automatically imply unrestricted authority.

For every destructive or sensitive operation verify:

- identity
- authorization
- resource scope
- action scope
- contextual constraints
- approval requirements where applicable

Test authorization boundaries independently from authentication.

---

## 7. Blast radius

Security review must quantify potential impact.

Ask:

- How many resources can this operation affect?
- Can it cross environments?
- Can it affect unrelated tenants?
- Can it affect other containers?
- Can it modify shared state?
- Can one compromised credential affect the whole fleet?
- Can an agent operate globally?
- Can a malformed selector match everything?

Prefer explicit scope.

Examples:

- one resource instead of fleet-wide
- one container instead of host-wide
- one API method instead of arbitrary API access
- one directory instead of filesystem-wide access

The smaller the authority boundary, the smaller the failure domain.

---

## 8. Empty, ambiguous, and unexpected selections

Never assume that an empty selection means "nothing to do."

In destructive automation, an empty result can be catastrophic if interpreted as:

"everything is no longer desired."

Test:

- zero matches
- one match
- many matches
- ambiguous match
- stale match
- malformed selector
- unexpected wildcard
- missing environment
- missing configuration

For destructive operations, ambiguity should normally fail closed.

Do not allow:

"Could not determine target"

to silently become:

"Operate on all targets."

---

