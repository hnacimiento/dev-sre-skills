# sre-bash — Reference: Preconditions, Logging, and State Files (sections 48-65)

This file is loaded on demand from `sre-bash/SKILL.md` §1a, whenever the script depends on preconditions, produces logs an operator relies on, or persists state. It is
not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-bash's own core has already
been read.

---

# 48. File Deletion

Deletion is a mutation.

Treat:

    rm

as potentially destructive.

Before deletion consider:

- exact target;
- empty variable;
- wildcard expansion;
- symlink behavior;
- permissions;
- expected scope;
- whether backup exists;
- whether deletion is reversible.

Never construct destructive paths from partially validated input.

Avoid dangerous constructs where an empty variable could change:

    "$BASE/$TARGET"

into a broader path than intended.

---

# 49. Empty Values Are Dangerous

An empty variable is not necessarily harmless.

Examples:

- empty secret;
- empty path;
- empty container name;
- empty resource list;
- empty API response;
- empty selection.

For critical values, distinguish:

    unset

from:

    empty

from:

    valid value.

Do not let an empty value silently acquire dangerous semantics.

---

# 50. Preconditions Must Run Before Mutation

Before changing the system, validate everything reasonably knowable:

- dependencies;
- permissions;
- target identity;
- credentials;
- expected files;
- expected versions;
- required tools;
- available storage;
- required network access;
- backup destination;
- recovery capability.

The goal is:

    fail before mutation whenever possible.

But do not pretend preflight can predict every runtime failure.

Runtime verification remains necessary.

---

# 51. Do Not Overtrust Preflight

A precheck is a snapshot.

The world can change afterward.

Examples:

- container disappears;
- permissions change;
- file changes;
- network disappears;
- disk fills;
- API state changes;
- another process modifies the target.

Therefore:

    preflight protects against known invalid initial conditions.

It does not prove that the later operation will succeed.

---

# 52. Time-of-Check / Time-of-Use

Avoid assuming:

    check X
    →
    later use X

means X remains unchanged.

If a property matters at mutation time, verify or enforce it at the point of
use.

Locks, atomic operations, transactional APIs, and identity verification may
be required.

---

# 53. Logging

Logs should make operational state understandable.

Record useful facts such as:

- operation identifier;
- target;
- phase;
- resource;
- action;
- result;
- verification;
- recovery state.

Do not log secrets.

Avoid noisy output that hides the important failure.

Prefer:

    RESOURCE=foo RESULT=ATTEMPT_FAILED REASON=HASH_MISMATCH

over:

    ERROR!!!

The exact format may vary.

The principle is machine- and human-useful observability.

---

# 54. Error Messages Must Be Actionable

A useful error should communicate:

    what failed
    where
    during which phase
    why it failed
    what state may remain
    what recovery was attempted
    what the operator should inspect

Avoid messages that only report:

    command failed

when more information is available.

Do not claim:

    "rollback completed"

when only the rollback command was started.

---

# 55. Preserve the Primary Failure

A cleanup or recovery failure should not erase the original failure.

Example:

    primary failure:
        docker cp failed

    recovery failure:
        restore of JS #4 failed

The final result should preserve both facts.

Do not report only:

    cleanup failed

and lose the actual reason the operation entered recovery.

---

# 56. Primary Result vs Recovery Result

Keep these concepts separate.

Example:

    INSTALL_RESULT=FAILED
    ROLLBACK_RESULT=SUCCESS

This means:

    installation failed,
    recovery succeeded.

It does NOT mean:

    overall operation succeeded.

Recovery cannot magically turn a failed primary operation into success.

This distinction is essential for monitoring and incident analysis.

---

# 57. Retry Carefully

Retries are not automatically reliable.

Before retrying ask:

- Is the operation idempotent?
- Can the first attempt have partially succeeded?
- Could retry duplicate side effects?
- Is the error transient?
- Is the resource changing?
- Is the timeout masking a slow successful operation?

Retry only when semantics support it.

Prefer bounded retries with:

- explicit count;
- delay/backoff where appropriate;
- clear logging;
- final failure classification.

---

# 58. Verification Retries

Verification can itself fail transiently.

For example:

    mutation succeeded
    verification GET temporarily failed

If appropriate, retry verification before declaring failure.

But do not retry indefinitely.

Define:

    retry budget
    timeout
    final classification.

Do not turn uncertainty into false success.

---

# 59. Network Operations

For `curl` and similar commands:

- use explicit timeouts;
- distinguish connection failure from HTTP failure;
- inspect HTTP status;
- validate response structure;
- avoid leaking credentials;
- retry only safe operations;
- verify semantic state after mutation where required.

A successful POST is not necessarily proof that the desired server state exists.

For critical mutations:

    POST
    →
    GET
    →
    verify expected state

may be necessary.

---

# 60. Environment and Portability

Know what environment the script expects.

Consider:

- Bash version;
- GNU vs BSD utilities;
- locale;
- PATH;
- shell invocation;
- permissions;
- filesystem behavior;
- container image;
- operating system.

If portability matters, test the supported environments.

Do not claim portability that was not tested.

---

# 61. Dependency Detection

Detect required commands before mutation.

Examples:

    docker
    curl
    jq
    sha256sum
    flock
    shellcheck

Do not discover a missing critical dependency after partially mutating production.

Where practical, verify versions when behavior is version-sensitive.

---

# 62. Lock Files Are State, Not Locks

A file named:

    .lock

does not create synchronization merely because it exists.

Use actual locking primitives.

If a file is also used to record ownership or metadata, distinguish:

    kernel lock

from:

    informational state file.

Do not implement concurrency protection with:

    test -f lock

followed by:

    touch lock

without understanding the race.

---

# 63. Shared State Outside the Lock

If the primary resource is protected by a per-resource lock, inspect all
shared state separately.

For example:

    target-specific lock
    +
    global state file

can still race.

A shared state file should have:

- an appropriate synchronization strategy;
- atomic writes;
- well-defined ownership;
- recovery from incomplete writes.

Atomicity and locking solve different problems.

---

# 64. State File Design

A persistent state file should have:

- explicit schema;
- versioning if useful;
- validation;
- atomic update;
- clear ownership;
- safe recovery from corruption.

Do not assume a state file is valid merely because it exists.

Treat state files as data with contracts.

---

# 65. Temporary State vs Durable State

Know which state can be lost safely.

### Temporary state

Loss is acceptable.

### Recovery-critical state

Loss can prevent recovery.

### Operational state

Loss may produce incorrect future decisions.

The storage strategy must match the importance of the state.

Do not treat every file under `$SCRIPT_DIR` as equivalent.

---

