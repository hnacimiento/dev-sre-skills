# sre-testing — Reference: Security, Observability, Time, Boundary, and Destructive-Operation Testing (sections 15-20a)

This file is loaded on demand from `sre-testing/SKILL.md`'s load-on-demand
index, when testing security-relevant behavior, observability, exit codes, retries, boundaries, or destructive/dry-run operations. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-testing's own core has already been read.

---

## 15. Security testing

Security controls must be tested against the actual threat they claim to prevent.

Examples:

If the contract says:

"secret must not appear in artifact"

test:

- secret absent
- secret present
- secret-like placeholder present
- variable name present
- empty secret
- malformed secret
- secret passed through command arguments
- secret exposed in logs
- secret exposed in generated files

Do not test only the string representation of the variable.

Test the actual sensitive value and its possible exposure paths.

## 16. Observability testing

A failure that cannot be diagnosed is operationally expensive.

Tests should verify that failures produce enough evidence to answer:

- What operation was running?
- Which resource failed?
- Why did it fail?
- What had already changed?
- What was recovered?
- What was not recovered?
- What should the operator do next?
- What is the exit status?
- Where are recovery artifacts?
- Can the result be distinguished from success?

Test logs and summaries as part of the operational contract.

Do not require operators to reconstruct state from raw debug output.

## 17. Exit-code testing

Exit codes are part of the API of automation.

For every meaningful result test:

- stdout/stderr
- exit code
- resulting state

Never test exit code independently of system state.

Examples:

SUCCESS should not merely mean:

"last command returned 0."

DEGRADED should only be used when degradation is known and explicitly characterized.

PARTIAL recovery must not silently become success.

A successful recovery must never erase the fact that the primary operation failed.

## 18. Time and retry behavior

Test:

- immediate retry
- delayed retry
- repeated retry
- timeout
- dependency recovery
- transient verification failure
- permanent verification failure

Retries must not create duplicate mutations or amplify failure.

Every retry needs a reason.

Do not blindly retry destructive operations.

## 19. Boundary testing

Test:

- empty input
- one item
- maximum expected items
- malformed item
- missing item
- duplicate item
- unexpected extra item

For scripts operating on collections, explicitly test:

- zero resources
- one resource
- all resources
- partially matching resources

Never assume an empty result is safe.

For destructive operations, an empty selection must be treated as a potentially dangerous state and tested explicitly.

## 20. Destructive operations

Destructive automation requires a higher testing bar.

Test:

- correct target
- wrong target
- empty target
- stale target
- ambiguous target
- partially matching target
- target disappears
- dependency changes between selection and mutation

For destructive actions ask:

"Could a valid execution of this code destroy the wrong thing?"

Do not rely solely on unit tests.

Use realistic staging or fault-injection environments.

Name one destructive pattern specifically, because it is more dangerous
than an explicit delete command and easy to under-test: turn-down by
absence, where a resource is destroyed not because it was explicitly
targeted but because it is missing from a desired-state list (a
reconciliation loop that deletes "anything not in the current config").
Under this pattern, any failure that produces an incomplete or empty
desired-state list — a parser bug, a truncated read, a stale cache, an
authentication failure that silently returns zero entries — becomes an
implicit order to delete everything. Test this pattern with its own
scenarios: a parser that silently drops entries, a config source that
returns a truncated list, a source that fails and returns empty rather
than erroring, and a source that returns stale data. In every case verify
that the system distinguishes a confirmed, successfully-queried empty
state from a failed or incomplete one, and treats the latter as unsafe to
act on rather than as "nothing desired."

## 20a. Dry-Run Testing

Do not trust a `--dry-run` flag by name. Prove it does what it claims.

At minimum, test:

- **State comparison.** Capture the full relevant state before and after
  the dry-run invocation and assert it is byte-for-byte or semantically
  identical. A dry-run that prints "would delete 42 servers" but never
  demonstrates that zero servers were actually touched is an unverified
  claim, not a tested property.
- **Every layer, not just the entry point.** In a system with multiple
  layers (CLI, planner, cache, external API, mutator), a `--dry-run` flag
  handled only at the CLI layer does not prove the layers underneath are
  side-effect-free. Test each layer independently for mutation during a
  dry-run invocation — does the planner still write to a cache; does the
  call reach the external API at all; does the mutator receive and
  silently discard a request, or never get invoked?
- **Selection logic parity.** Confirm the dry-run evaluates the same
  target-selection logic the real run would use, not a simplified or
  stale approximation — a dry-run that reports on a different resource
  set than the real run would touch gives false confidence.
- **Read-only side effects.** A dry-run that only avoids mutating the
  primary target can still write local logs, temp files, or a cache; that
  may be acceptable, but it should be an explicit, tested, documented
  property rather than an accidental discovery.
- **Concurrent drift.** If state can change between a dry-run preview and
  the real run that follows it, test whether the tool re-validates at
  execution time or blindly trusts the preview.

