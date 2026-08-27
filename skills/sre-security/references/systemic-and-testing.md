# sre-security — Reference: System-Level Security and Testing (sections 35-51)

This file is loaded on demand from `sre-security/SKILL.md` §1a, whenever reasoning about system-level interactions, security testing, or closing out a security review. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-security's own core has already
been read.

---

## 35. System-level interactions

Do not assume secure components produce a secure system.

Analyze interactions.

Example:

Component A:
validates target.

Component B:
resolves target again.

Component C:
uses cached identity.

Component D:
performs mutation.

Each component may be correct independently.

The system can still be unsafe if the identity observed by A differs from the identity mutated by D.

Test:

- stale state
- race conditions
- TOCTOU
- identity changes
- target replacement
- authorization changes between validation and mutation
- dependency disagreement

Security properties must survive system dynamics.

---

## 36. TOCTOU

Whenever the system:

1. checks something
2. waits
3. acts on it

consider whether the checked state can change.

Examples:

- file existence
- permissions
- container identity
- resource ownership
- authorization
- deployment version
- API state

Ask:

"Can the world change between the check and the action?"

If yes, determine whether the design needs:

- atomic operation
- transaction
- locking
- revalidation
- immutable identifier
- postcondition verification

---

## 37. Replay and stale authorization

Sensitive operations may be replayed.

Test:

- repeated authorization
- old approval
- old token
- stale recovery artifact
- stale deployment artifact
- repeated rollback
- repeated destructive operation

An operation that was valid yesterday may not be valid today.

Authorization should be evaluated in the context in which the mutation occurs.

---

## 38. Dependency compromise

Do not model dependencies only as unavailable.

Consider:

- compromised dependency
- malicious response
- unexpected response
- validly authenticated but malicious content
- dependency returning attacker-controlled configuration

Authentication does not automatically imply content safety.

Validate what the dependency returns.

---

## 39. Security observability

Security failures must be diagnosable without exposing sensitive data.

Test whether logs reveal:

- failed authorization
- unexpected target
- integrity mismatch
- privilege escalation
- suspicious input
- agent policy violation
- blocked action

But verify that logs do not reveal:

- credentials
- tokens
- private keys
- sensitive payloads

The absence of secrets in logs is itself a security invariant.

---

## 40. Security testing strategy

Do not test only vulnerabilities.

Test security properties under:

- normal operation
- degraded operation
- recovery
- concurrency
- interruption
- stale state
- malicious input
- dependency failure
- dependency compromise
- operator error
- agent error

Security testing should intersect reliability testing.

---

## 41. Security regression discipline

Every discovered security or reliability-security defect should be evaluated for permanent regression coverage.

If a bug allowed:

- secret exposure
- unauthorized mutation
- incorrect target selection
- privilege escalation
- unsafe recovery
- integrity bypass

create a regression test that captures the underlying failure class.

Do not rely on human memory.

---

## 42. Security review questions

Before approving an automation or operational system, ask:

### Authority

Who can invoke it?

Who can authorize it?

What can it modify?

### Scope

What is the maximum blast radius?

Can an empty or ambiguous selection expand scope?

### Secrets

Where do secrets exist?

Where can they leak?

### Integrity

How do we know artifacts are authentic and unchanged?

### Recovery

Can recovery introduce malicious or stale state?

### Concurrency

Can authorization or target identity change during execution?

### Operator

Can a stressed operator accidentally perform the dangerous action?

### Agent

Can probabilistic reasoning cross a deterministic authority boundary?

### Adversary

What happens if production data is malicious?

### Failure

What happens if a security dependency disappears?

### Observability

Can we investigate without exposing additional secrets?

---

## 43. Security anti-patterns

Treat these as warning signs:

- unrestricted root automation
- arbitrary `sh -c`
- `eval`
- secrets in command-line arguments
- secrets in generated scripts
- trusting cached identity for privileged mutation
- treating variable names as proof that secret values are absent
- treating HTTP 200 as proof of secure state
- treating exit 0 as proof of desired state
- trusting downloaded "latest" artifacts without integrity/provenance controls
- destructive operations on empty selections
- broad agent tools
- model-controlled authorization
- prompt-based security boundaries
- security checks performed only before mutation
- recovery without integrity verification
- logs containing sensitive material
- relying on cleanup traps for secret protection
- assuming authentication equals authorization
- assuming secure components imply secure system behavior

Do not automatically declare an anti-pattern catastrophic.

Evaluate:

- context
- exposure
- privilege
- blast radius
- compensating controls
- detectability
- recoverability

---

## 44. Security and failure reporting

A security-relevant failure must be observable.

Do not allow:

warning-only behavior

when the security invariant was actually violated.

Distinguish:

- blocked unsafe operation
- security control unavailable
- security control failed
- security invariant violated
- uncertain security state

"Could not verify security"

must not become:

"security verified."

Uncertainty is itself an operational state.

---

## 45. Security versus recovery

Never allow recovery to bypass security controls merely because the system is already degraded.

Recovery may require elevated authority, but it must still respect:

- authorization
- scope
- integrity
- auditability
- secret handling

Emergency does not mean unrestricted.

If emergency access exists, define and test it explicitly.

---

## 46. Security testing of generated recovery artifacts

Generated rollback/recovery scripts require independent security review.

Verify:

- generated permissions
- embedded paths
- embedded credentials
- target selection
- shell expansion
- command construction
- source integrity
- backup integrity
- recovery authorization
- post-restore verification

Run the generated artifact under realistic failure conditions.

Do not assume the security properties of the generator automatically transfer to the generated artifact.

---

## 47. Supply-chain change detection

If the system intentionally consumes changing upstream artifacts, document that the model is:

"change detection"

rather than:

"strong reproducibility."

Security review should then ask:

- what changed?
- when?
- from where?
- was the change expected?
- can the operator identify the exact artifact?
- can the previous artifact be recovered?

Do not hide mutable dependencies behind language suggesting immutable provenance.

---

## 48. Empirical security validation

Security assumptions should be tested empirically.

Examples:

- inject malicious log content
- alter downloaded artifacts
- corrupt backups
- race authorization checks
- interrupt writes
- run concurrent operations
- simulate compromised dependencies
- simulate operator mistakes
- attempt unauthorized agent tool calls

A control that exists only in documentation is not evidence.

A control that has never been exercised under failure is weak evidence.

---

## 49. Security definition of done

Security-sensitive automation is not complete merely because:

- secrets are not visible in the final file
- authentication works
- unit tests pass
- lint passes
- the normal deployment succeeds

A stronger definition of done is:

- threat model exists
- assets identified
- trust boundaries identified
- authority scoped
- blast radius understood
- secrets lifecycle reviewed
- integrity verified
- destructive operations tested
- ambiguous selections tested
- recovery security tested
- concurrency considered
- interruption considered
- auditability verified
- security failures observable
- regression tests exist
- generated artifacts tested
- agent boundaries enforced where applicable
- operator behavior considered

---

## 50. Final SRE security question

Before declaring the system secure enough to operate, ask:

"If an attacker controls an unexpected input, an operator makes a reasonable mistake, a dependency behaves maliciously, an automation path fails partially, or an AI agent reasons incorrectly, what is the maximum damage that can still occur?"

Then ask:

"Which deterministic boundaries prevent that damage?"

And finally:

"How have we empirically demonstrated that those boundaries actually hold?"

If the answer depends on:

- trusting the operator
- trusting the prompt
- trusting the model
- trusting cached state
- trusting exit 0
- trusting HTTP 200
- trusting an upstream artifact
- trusting a rollback
- trusting cleanup traps
- trusting documentation

the security design is not finished.

---

## 51. A Working Definition of "Sufficient"

Close every security review with a definition that does not imply
absolute safety, because no review can honestly claim that. A useful
working definition:

    Security is sufficient for operation when every deterministic
    boundary standing between an adversarial input, a reasonable
    operator mistake, or a partial failure and catastrophic impact
    has been explicitly identified, has been exercised under realistic
    adversarial and failure conditions rather than merely described,
    and is monitored well enough that its silent failure would itself
    be detected.

This does not claim that no successful attack or failure is possible. It
claims that the boundaries the system is actually relying on are known,
tested, and observable rather than assumed. A system that cannot state
which boundaries it is relying on, or has never exercised them under
failure, has not met this bar regardless of how many individually correct
