# sre-testing — Reference: Bash, Generated-Code, Recovery, and Concurrency Testing (sections 8-14)

This file is loaded on demand from `sre-testing/SKILL.md`'s load-on-demand
index, when testing Bash scripts, generated code, recovery mechanisms, idempotency, concurrency, or interruption. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-testing's own core has already been read.

---

## 8. Bash-specific testing

For Bash automation, test at minimum:

- `set -e` behavior
- `set -u` behavior
- `pipefail`
- traps
- signal handling
- exit codes
- subshell behavior
- command substitution
- pipelines
- functions invoked under `||`
- functions invoked conditionally
- cleanup behavior
- temporary-file handling
- quoting
- filenames containing spaces
- filenames containing shell metacharacters
- empty variables
- missing variables
- empty command output
- unexpected command output
- partial writes
- interrupted writes
- lock acquisition
- lock contention
- generated shell scripts
- heredoc expansion
- runtime versus generation-time variable expansion

Never assume `set -e` provides reliable transactional semantics.

Test the actual control flow.

## 9. Generated-code testing

Generated scripts are independent software artifacts.

If a program generates:

- rollback scripts
- migration scripts
- deployment scripts
- recovery scripts
- configuration files
- shell fragments

test both:

1. the generator
2. the generated artifact

Minimum validation for generated Bash:

- `bash -n`
- ShellCheck when applicable
- representative execution
- failure-path execution
- exit-code verification
- generated content inspection

Do not assume that because the generator passes tests, the generated script is correct.

## 10. Recovery testing

Recovery is a first-class operation.

For every recoverable mutation test:

- recovery succeeds
- recovery partially succeeds
- recovery completely fails
- recovery is interrupted
- recovery dependency is unavailable
- recovery source is missing
- recovery source is corrupted
- target disappears during recovery
- verification after recovery fails

A recovery routine must never claim success that it cannot demonstrate.

If the system exposes:

SUCCESS
PARTIAL
FAILED
NOT_NEEDED

test the distinction between them explicitly.

Do not allow:

"recovery attempted"

to become equivalent to:

"recovery succeeded."

## 11. Resource-level outcomes

For multi-resource operations, test resources independently.

If an operation modifies:

- A
- B
- C
- D

inject failure into B and verify:

- A outcome is reported correctly
- B outcome is reported correctly
- C is still evaluated when safe
- D is still evaluated when safe
- aggregate result is correct

One failed resource must not hide the state of the remaining resources unless continuing would itself be unsafe.

Test for silent omission.

## 12. Idempotency

Run important operations repeatedly.

At minimum:

- first execution
- second execution
- third execution

Verify:

- no duplicated configuration
- no duplicated files
- no unnecessary mutations
- stable resulting state
- stable hashes where applicable
- no accumulation of unintended artifacts
- same meaningful outcome

For recovery operations, test repeated recovery as well.

Ask:

"What happens if the operator reasonably retries this command?"

Retries are normal operational behavior.

## 13. Concurrency

Assume operators and automation can overlap.

Test:

- same target concurrently
- different targets concurrently
- lock contention
- lock release after failure
- lock release after SIGTERM
- lock release after process death
- shared-state races
- atomic versus non-atomic writes
- stale state reads

A lock is not automatically proof of concurrency safety.

Test the scope of the lock.

Test what remains outside the lock.

## 14. Interruption testing

Interrupt operations at meaningful mutation boundaries.

Test:

- before mutation
- during mutation
- after partial mutation
- during verification
- during rollback
- during cleanup
- during state-file writes

For each interruption determine:

- what state remains
- whether recovery is possible
- whether recovery artifacts exist
- whether the exit status is meaningful
- whether the operator knows what happened

SIGKILL and power loss cannot be trapped.

Therefore test the artifacts and state that must make manual recovery possible afterward.

