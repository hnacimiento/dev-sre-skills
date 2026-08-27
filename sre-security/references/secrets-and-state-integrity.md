# sre-security — Reference: Secrets, State, and Integrity (sections 9-21)

This file is loaded on demand from `sre-security/SKILL.md` §1a, whenever secrets, generated scripts, supply chain, or recovery/backup integrity are involved. It
is not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-security's own core has already
been read.

---

## 9. Secrets

Treat secrets as high-risk data.

Review where secrets exist:

- memory
- environment
- files
- temporary files
- command arguments
- process listings
- logs
- stdout
- stderr
- generated scripts
- generated configuration
- backups
- crash dumps
- CI artifacts
- shell history

Ask:

"Where can this secret exist during the complete lifecycle of the operation?"

A secret is not safe merely because the final artifact does not contain it.

---

## 10. Secret validation

When a control claims:

"secret is not present"

verify the actual secret value.

Do not search only for:

- variable names
- symbolic placeholders
- labels
- configuration keys

Example:

Searching for:

EMBY_API_KEY

does not prove that the actual API key is absent.

Tests should include:

- variable name present
- placeholder present
- actual secret present
- empty secret
- malformed secret
- secret embedded in generated content

If the secret can be empty, explicitly define what that means.

Never allow an empty pattern to accidentally match everything.

A related and distinct failure: a secret value that happens to look like a
command-line flag or option to the tool checking for it (for example a
secret whose value is literally `-n`, `--`, or starts with a dash). Naive
detection commands such as `grep "$SECRET" file` or `echo "$SECRET"` can
silently misinterpret that value as an option rather than as search text
or output, producing a false negative in a security check that appears to
run successfully. This is a different bug class from searching by name
instead of value (section 10 above) and from an empty secret: it is about
the value's syntax colliding with the tool's own argument parsing. Treat
secret values as opaque data passed via `--`, a here-string, or an
explicit `-e`/`-F` style flag where the tool supports it, never as a bare
positional argument assembled by string interpolation.

---

## 11. Secret exposure through process arguments

Command-line arguments may be observable by:

- process inspection
- monitoring systems
- debugging tools
- other users
- container tooling

Avoid placing sensitive values in arguments when a safer mechanism exists.

Review commands such as:

- curl
- docker exec
- kubectl exec
- ssh
- cloud CLIs

Ask:

"Who can observe this command while it is running?"

Security controls should not introduce a larger exposure surface than the problem they are solving.

Do not assume that moving a secret-bearing command from the host into a
container automatically improves security. Inside the container, the
same class of exposure can still occur: `docker exec` argument lists are
visible to anyone who can inspect the container's process table or use
the container tooling; container logs and debugging sidecars can capture
the same command line; and anyone with `exec` access to the container has
at least as much visibility as a host user with process-inspection
rights. Moving the secret across that boundary changes who can observe
it, not whether it is observable. Evaluate exposure at the container
boundary with the same rigor as at the host boundary, rather than
treating "it's inside the container now" as a resolved finding.

---

## 12. Temporary files and cleanup

Temporary files containing sensitive information require lifecycle analysis.

Verify:

- secure permissions
- location
- ownership
- cleanup
- behavior on failure
- behavior on SIGTERM
- behavior on SIGKILL
- behavior after crash
- persistence in backups
- persistence in artifacts

Remember:

SIGKILL cannot execute cleanup traps.

Therefore secrets must not rely exclusively on cleanup traps for protection.

Prefer minimizing secret persistence in the first place.

---

## 13. Atomic state and security

State corruption can become a security issue.

A partially written state file may cause:

- incorrect target selection
- missing authorization context
- fallback to unsafe defaults
- execution against the wrong environment
- stale credentials
- incorrect recovery behavior

For shared state ask:

- Is the write atomic?
- Is it protected against concurrent writers?
- Can it be partially written?
- What happens after process death?
- What happens after power loss?
- Can stale state be mistaken for trusted state?

Atomicity is a security property when state influences authorization or target selection.

---

## 14. Trusted identity versus cached state

Cached state should not automatically be trusted as authoritative identity.

Examples:

- cached container name
- cached endpoint
- cached environment
- cached account
- cached deployment identifier

If cached state selects a privileged target, revalidate it against an authoritative source before performing sensitive mutations.

A cache is a convenience mechanism.

It should not silently become an authority boundary.

---

## 15. Input validation

Validate input at the boundary.

Do not assume inputs are safe because they originate from:

- configuration files
- administrators
- internal APIs
- CI systems
- repositories
- monitoring systems
- logs

Validate:

- type
- format
- range
- allowed values
- expected cardinality
- resource identity
- environment
- encoding

Reject ambiguity where the consequence is destructive.

---

## 16. Shell injection

Bash automation has a large injection surface.

Review:

- command substitution
- `eval`
- `sh -c`
- `bash -c`
- `docker exec`
- `ssh`
- `xargs`
- `find -exec`
- `sed`
- `awk`
- `grep`
- dynamic paths
- dynamically generated scripts
- heredocs

Never assume quoting alone proves safety.

Trace the origin of the value.

Ask:

"Can an attacker influence this value?"

Then:

"Where is this value interpreted as code?"

Data crossing into a command interpreter is a trust-boundary transition.

---

## 17. Generated scripts are security boundaries

If a program generates:

- rollback scripts
- deployment scripts
- migration scripts
- recovery scripts

the generated artifact must be treated as executable code.

Review:

- variable expansion
- escaping
- permissions
- embedded secrets
- embedded paths
- embedded credentials
- attacker-controlled values
- command substitution
- heredoc expansion

Test the generated artifact independently.

A secure generator can produce an insecure generated script.

---

## 18. Supply chain

Security analysis must include everything the automation consumes.

Examples:

- downloaded files
- packages
- containers
- Git repositories
- release artifacts
- JavaScript addons
- shell scripts
- APIs
- remote configuration

Ask:

- Is the source authenticated?
- Is integrity verified?
- Is provenance known?
- Is the version pinned?
- Is "latest" being trusted?
- Is change detected?
- Can a compromised upstream source modify production behavior?
- Can the artifact contain credentials or malicious code?

Distinguish:

"we detected that something changed"

from:

"we can prove exactly which artifact we intended to execute."

Change detection is useful.

Strong reproducibility and provenance are stronger guarantees.

Do not claim one when only the other exists.

An integrity check is only as strong as the independence of where the
expected value is stored. A SHA256 checksum stored in the same
repository, the same deployment artifact, or otherwise the same trust
domain as the downloaded artifact it is meant to validate provides no
protection against an attacker capable of modifying that trust domain:
they can update the artifact and the stored hash together. Verifying
integrity meaningfully requires the expected value to originate from a
domain the attacker's access does not also cover — a separate signing
authority, a pinned value reviewed and merged through a different path,
or an external attestation service. Treat "we checked the hash" as
unproven until you can state where the expected hash lives and why that
location is not reachable by the same compromise that would need to be
detected.

---

## 19. Integrity verification

When an artifact is copied, downloaded, generated, or restored, verify integrity where required.

Examples:

- SHA256
- signature
- version
- expected metadata
- semantic configuration state
- API read-back

Do not rely exclusively on:

- HTTP 200
- command exit 0
- successful file copy

The relevant question is:

"Does the resulting artifact match the trusted artifact?"

---

## 20. Security of recovery

Recovery paths are privileged paths.

Do not assume rollback is inherently safe.

Review:

- rollback credentials
- backup integrity
- backup confidentiality
- recovery target selection
- recovery authorization
- generated recovery scripts
- stale backups
- malicious backups
- partial restoration
- recovery verification

A compromised rollback artifact can be as dangerous as a compromised deployment artifact.

Recovery must preserve the same security invariants as forward execution.

---

## 21. Backups

Backups contain valuable state.

Review:

- permissions
- ownership
- encryption where appropriate
- retention
- deletion
- integrity
- provenance
- secret content
- access control

Ask:

"If the backup is stolen, what does it reveal?"

And:

"If the backup is modified, could recovery deploy malicious state?"

Backup integrity and backup confidentiality are separate properties.

There is a third, distinct failure that integrity checks alone do not
catch: a backup can be entirely unmodified since its creation and still
faithfully preserve an already-compromised state, if it was captured
after an attacker's change and before that change was detected. A rollback
that restores from such a backup returns exit 0, passes a hash check
against itself, and reintroduces the compromise as if it were the trusted
prior state. Verifying "this backup was not tampered with after creation"
is not the same claim as "this backup predates the earliest known
compromise." When recovery matters for security, not just for
availability, establish and record backup timing relative to detection
and known-good checkpoints, not only backup integrity from its own
creation time forward.

---

