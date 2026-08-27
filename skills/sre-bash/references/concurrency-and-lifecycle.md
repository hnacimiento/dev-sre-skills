# sre-bash — Reference: Idempotency, Concurrency, and Process Lifecycle (sections 19-30)

This file is loaded on demand from `sre-bash/SKILL.md` §1a, whenever the script can run more than once, concurrently, or must survive signals. It is
not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-bash's own core has already
been read.

---

# 19. Idempotency

Ask three questions:

    What happens if it succeeds and runs again?

    What happens if it fails halfway and runs again?

    What happens if recovery runs twice?

Prefer operations that converge toward the desired state.

But do not assume idempotency merely because a command can be repeated.

For destructive operations, explicitly define repeated behavior.

A script that repeatedly performs the wrong mutation is still wrong.

---

# 20. Temporary Files

Use temporary files deliberately.

Temporary files should have:

- predictable lifecycle;
- appropriate permissions;
- cleanup behavior;
- secure location;
- unique names;
- clear ownership.

Prefer mechanisms such as `mktemp` rather than predictable filenames.

Do not place secrets in world-readable temporary files.

Do not assume cleanup traps will always execute.

`SIGKILL` and power loss exist.

For important state, design the system so incomplete cleanup is survivable.

---

# 21. Atomic File Updates

When updating persistent state:

    write temporary complete state
       ↓
    verify if appropriate
       ↓
    atomically rename into place

Prefer:

    mktemp
    write
    mv

over:

    printf ... > statefile

when a partially written state file would be dangerous.

Atomicity applies to the state representation, not merely to the shell command.

Ask:

    "What does another process see if this process dies halfway through?"

The desired answer is usually:

    old valid state
    OR
    new valid state

not:

    truncated state.

---

# 22. Atomicity Has Limits

`mv` is useful for atomic replacement on the same filesystem.

It does not magically provide:

- distributed transactions;
- multi-file atomicity;
- rollback;
- crash consistency across arbitrary storage;
- synchronization with external systems.

If several files must change consistently, a single atomic rename may not be enough.

Model the entire state transition.

---

# 23. Concurrency

Assume multiple invocations may overlap unless explicitly prevented.

Ask:

    "Can two copies of this script run simultaneously?"

If not, enforce it.

Prefer kernel-backed synchronization such as:

    flock

over:

    if [ -f lock ]; then ...

because existence checks are race-prone.

Locks should cover the actual critical section.

Define lock scope explicitly.

---

# 24. Lock Scope

A global lock may unnecessarily serialize independent operations.

A lock that is too narrow may permit corruption.

Choose scope based on the shared resource.

For example:

    same target resource
        → same lock

    independent target resources
        → may run concurrently

Also identify shared state outside the target-specific lock.

A script can correctly lock its primary resource while still racing on:

- global cache files;
- shared state files;
- logs;
- generated indexes;
- metadata.

Concurrency analysis must include all shared mutable state.

---

# 25. Signals

At minimum, reason about:

    SIGINT
    SIGTERM
    ERR
    EXIT

depending on the script.

Do not assume:

    trap = guaranteed cleanup.

Signals can arrive:

- before mutation;
- during mutation;
- during recovery;
- during cleanup;
- during verification.

A signal handler must itself be reliable.

Avoid complicated logic inside traps.

Keep traps small and delegate to controlled functions.

---

# 26. Trap Reentrancy

Multiple failure paths can converge on the same cleanup or recovery routine.

Examples:

    ERR
    INT
    TERM

may all reach recovery.

Prevent recovery from executing twice.

Use an explicit state such as:

    RECOVERY_RUNNING=1

or an equivalent mechanism.

The exact implementation is not important.

The invariant is:

    recovery must be single-entry unless explicitly designed otherwise.

---

# 27. Do Not Let Recovery Trigger Recovery Recursively

Recovery commands can fail.

If the recovery function itself triggers the same `ERR` trap that started it,
you can create:

    failure
      ↓
    recovery
      ↓
    failure
      ↓
    recovery
      ↓
    ...

Disable or contain recursive recovery deliberately.

Do not simply suppress all errors globally.

Contain the failure at the recovery boundary.

---

# 28. `SIGKILL` Is Different

`SIGKILL` cannot be trapped.

Neither can power loss.

Therefore:

    trap-based recovery != guaranteed recovery.

For operations that must survive abrupt process death:

- persist enough state before mutation;
- generate recovery artifacts before mutation;
- maintain backups;
- make operations restartable;
- make recovery externally executable;
- make partial state detectable.

The system must be recoverable without relying on the dead process.

---

# 29. Cleanup Is Not Recovery

Do not confuse:

    cleanup

with:

    rollback.

Cleanup may mean:

- delete temp files;
- close descriptors;
- remove transient artifacts.

Recovery means:

- restore an acceptable system state.

A script can successfully clean up its temporary directory while leaving
production partially mutated.

Report these separately.

The inverse also needs an explicit rule: a cleanup failure occurring after
the primary mutation has already succeeded and been verified must not
silently downgrade a verified SUCCESS into a reported FAILURE. Losing a
temporary file or failing to remove a scratch directory is a different
severity of problem than an unverified or partially failed mutation.
Represent them as distinct fields (for example
`PRIMARY_RESULT=SUCCESS, CLEANUP_RESULT=FAILED`) rather than collapsing
them into a single exit code that forces the operator to guess which one
actually happened.

---

# 30. `source` Is Executable Input

Treat:

    source file

as execution of code.

A sourced file may:

- fail;
- exit;
- modify variables;
- alter shell options;
- define traps;
- modify functions;
- execute arbitrary commands;
- contain syntax errors.

Do not treat configuration sourcing as a passive read.

For critical recovery paths, handle source failures explicitly.

If a required secrets/config file cannot be loaded:

    classify the affected operation as failed.

Do not let `set -e` silently terminate the function before the final operational
state is computed.

---

