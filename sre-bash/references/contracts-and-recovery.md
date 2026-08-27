# sre-bash — Reference: Contracts, Failure, and Recovery (§2–§18a)

This file is loaded on demand from `sre-bash/SKILL.md` §1a, whenever the script performs any consequential mutation at all. It is
not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-bash's own core (including its
§1 "Bash Is Not the System" — shell correctness != operational
correctness, exit 0 != desired state verified) has already been read.
Everything below builds on that §1 without repeating it.

---

# 2. Start With the Operational Contract

Before implementing a consequential Bash script, define:

### Inputs

- command-line arguments;
- environment variables;
- configuration files;
- secrets;
- filesystem state;
- external resources.

### Preconditions

What must be true before mutation?

### Mutations

What exactly can the script change?

### Postconditions

What must be true when the script claims success?

### Failure states

What partial states can exist?

### Recovery

How can the system return to a known acceptable state?

### Exit contract

What does each exit code mean?

### Operator contract

What information does an operator need after success or failure?

Do not let these properties emerge accidentally from shell control flow.

---

# 3. `set -Eeuo pipefail` Is Not a Reliability Guarantee

Use:

    set -Eeuo pipefail

when appropriate.

But never treat it as proof that the script is safe.

It does not automatically provide:

- transactional behavior;
- rollback;
- atomicity;
- correct error classification;
- correct cleanup;
- correct recovery;
- postcondition verification;
- protection against partial mutation;
- protection against unsafe external commands;
- protection against incorrect logic;
- protection against signals;
- protection against `SIGKILL`;
- protection against false success.

Think of shell options as language semantics controls.

They are not an operational state machine.

---

# 4. `set -e` / `errexit` Requires Explicit Reasoning

`set -e` has context-sensitive behavior.

Do not assume:

    any non-zero command
    =
    entire script immediately stops.

Commands inside constructs such as:

- `if`;
- `while`;
- `until`;
- `&&`;
- `||`;
- pipelines;
- command substitutions;
- functions invoked in conditional contexts;

can have different `errexit` behavior.

A production script should not depend on subtle `errexit` propagation to
implement critical state transitions.

For consequential operations:

- explicitly capture important failures;
- explicitly classify them;
- explicitly update operational state;
- explicitly continue or abort;
- explicitly verify final state.

A common dangerous pattern is:

    command || true

This is acceptable only when the ignored failure is intentional and its
operational meaning is explicitly understood.

Never use `|| true` merely to silence a failure.

## 4a. `set -u` / Nounset Deserves the Same Scrutiny as `errexit`

`set -u` (nounset) is not a simpler, safer sibling of `set -e`. It has its
own sharp edges that a reviewer must reason about explicitly:

- `${VAR:-default}` and `${VAR:+alt}` expansions intentionally bypass
  nounset for that expression. Code guarded this way is not protected by
  `set -u` even though the option is enabled script-wide.
- Arrays behave inconsistently across shells and versions when empty:
  `"${arr[@]}"` on an empty array can trigger an unbound-variable error
  under `set -u` in some Bash versions/configurations. Do not assume this
  is portable without testing the actual target environment.
- A variable that is merely empty (`VAR=""`) is not the same as unset
  under `set -u`; nounset only catches the unset case. A destructive
  operation guarded only by `set -u` can still receive an empty string
  and behave dangerously (see section 49, Empty Values Are Dangerous).
- Sourcing a file that does not export an expected variable can surface
  as a nounset failure far from the actual root cause; the error location
  and the misconfiguration location are frequently different, which
  matters for how actionable the failure message is (section 54).
- `set -u` failing mid-function does not by itself tell you whether any
  mutation had already occurred; treat it exactly like any other abrupt
  failure for state-tracking purposes, not as a special "safe" failure
  mode.

Reviewing `set -u` usage means checking these specific behaviors against
the actual shell and version in use, not assuming the option provides a
blanket guarantee once it appears at the top of the script.

---

# 5. Error Suppression Must Have a Reason

Every suppressed error should answer:

    "Why is this failure safe to ignore?"

Good examples may include:

- best-effort cleanup of a temporary file;
- removing an already absent resource where absence is acceptable;
- preventing a secondary cleanup failure from replacing the primary failure.

Dangerous examples include:

- installation commands;
- backup commands;
- restore commands;
- integrity checks;
- security checks;
- authorization checks;
- state persistence;
- verification.

Do not suppress errors from operations that establish or verify safety.

---

# 6. Exit Codes Are an Operational API

Treat exit codes as part of the script's public contract.

A useful contract distinguishes at least:

    0 = primary operation succeeded and was verified

    non-zero = something requires attention

If the system supports a degraded state, define it explicitly.

For example:

    0 = SUCCESS
    1 = FAILED
    2 = DEGRADED

Do not invent exit codes without documenting their semantics.

More importantly:

    exit status must derive from the primary operational result.

Do not accidentally inherit the status of the last `echo`, `printf`, cleanup
command, or unrelated operation.

Avoid:

    echo "done"

being the final operation that implicitly determines success.

Use an explicit final status.

---

# 7. Exit Status Is Not State

A script may exit with:

    0

while having:

- partially mutated a system;
- failed to verify a resource;
- skipped a required operation;
- silently failed to restore a resource;
- left stale state;
- produced an incomplete artifact.

Therefore model operational state separately from shell return status.

Example conceptual model:

    INSTALL_RESULT=SUCCESS
    ROLLBACK_RESULT=NOT_NEEDED

The exact variables are implementation-specific.

The principle is:

    operational state != process status.

---

# 8. Define State Before Mutation

For consequential scripts, define explicit lifecycle states.

Example:

    INIT
      ↓
    PRECHECKED
      ↓
    STAGED
      ↓
    MUTATING
      ↓
    VERIFYING
      ↓
    SUCCESS

Failure states should distinguish:

    FAILED_NO_MUTATION

from:

    FAILED_AFTER_MUTATION

because the recovery requirements are different.

Do not infer whether mutation occurred merely from where execution appears to
have stopped.

Persist the relevant state when recovery may occur in another process.

---

# 9. The Mutation Boundary Must Be Explicit

There should be a clear point after which the script considers itself capable
of requiring recovery.

Before the mutation boundary:

- validate inputs;
- validate dependencies;
- detect target resources;
- validate credentials;
- validate artifacts;
- validate expected state;
- prepare backups;
- prepare recovery artifacts.

After the mutation boundary:

- assume partial state is possible;
- preserve recovery information;
- handle signals;
- verify every mutation;
- produce explicit recovery state.

Do not begin destructive work before recovery mechanisms are available.

---

# 10. Backup Before Mutation

If recovery requires a previous state, capture it before mutation.

A backup is useful only if:

- it exists before mutation;
- it corresponds to the actual target;
- it is readable;
- it is integrity-verifiable;
- its location is known;
- its lifecycle is understood.

Do not call something a backup merely because a copy command was attempted.

Prefer:

    backup created
    +
    backup verified
    =
    recovery source established.

---

# 11. Recovery Is Another Bash Program

Rollback is not a magical reversal.

Treat recovery as an independent operation with:

- preconditions;
- mutations;
- per-resource outcomes;
- verification;
- error handling;
- observability;
- exit status.

Recovery itself can fail.

Therefore define:

    SUCCESS
    PARTIAL
    FAILED

where appropriate.

A recovery function must not silently remain in a pre-invocation state after
it has started executing.

For example:

    NOT_NEEDED

should mean:

    "recovery was never required/invoked."

It should never mean:

    "recovery started but aborted before calculating its result."

## 11a. Recovery Can Share a Root Cause With the Failure It Responds To

Do not assume recovery runs in a healthier environment than the one that
just failed.

The condition that caused the primary failure often also degrades
recovery itself. Examples:

- disk full caused the primary write to fail; the rollback's own state
  file or backup write can fail for the identical reason;
- the credential that expired mid-operation is the same credential
  recovery needs to authenticate its own restore calls;
- the network partition that broke the forward API call is still present
  when recovery tries to call the same API to reverse it;
- the container that disappeared mid-mutation is the same container
  recovery expects to reconnect to.

A recovery design that has never been tested under the specific failure
condition that triggers it is unverified precisely where it matters most.
When reviewing recovery logic, explicitly ask whether the failure being
recovered from could also disable the recovery path itself, and whether
the script detects and reports that condition (for example, an explicit
`RECOVERY_BLOCKED_BY_SAME_CAUSE`-style state) rather than retrying blindly
or timing out silently.

---

# 12. Per-Resource Recovery State

When recovery operates on multiple resources, track each resource independently.

Conceptual outcomes:

    RESTORED
    REMOVED
    ATTEMPT_FAILED
    NOT_ATTEMPTED

Definitions:

### RESTORED

The expected backup was applied and the postcondition was verified.

### REMOVED

The resource was expected not to exist and its absence was verified.

### ATTEMPT_FAILED

The script attempted the operation but command execution or verification
failed.

### NOT_ATTEMPTED

The resource required action but execution never reached that resource.

`NOT_ATTEMPTED` is operationally significant.

Do not silently convert it into:

    "nothing to do."

---

# 13. Aggregate Recovery Honestly

For multiple resources:

    SUCCESS
        all required resources are RESTORED or REMOVED

    PARTIAL
        some resources succeeded and some failed or were not attempted

    FAILED
        no required resource was successfully recovered

The exact aggregation may vary by system.

The principle does not:

    recovery result must describe reality.

Do not allow one successful resource to hide five failed resources.

---

# 14. Failure Containment During Recovery

A failure processing resource A should not automatically prevent the script
from determining the state of resources B, C, and D when continuing is safe.

For example:

    A -> RESTORED
    B -> ATTEMPT_FAILED
    C -> RESTORED

is operationally more useful than:

    "rollback failed."

However, containment must be deliberate.

Ask:

    "Can processing the next resource safely continue?"

Never continue merely because the script can technically continue.

---

# 15. Postconditions Define Success

A command returning zero proves only that the command reported success.

It does not prove that the desired state exists.

Examples:

    docker cp -> exit 0
    does not prove
    destination == expected content

    curl POST -> HTTP 200
    does not necessarily prove
    server state == desired configuration

    mkdir -> exit 0
    does not prove
    permissions == expected permissions

Therefore:

    command success
        ↓
    postcondition verification
        ↓
    operational success

Whenever practical, verify:

- existence;
- absence;
- size;
- permissions;
- ownership;
- hash;
- content;
- API state;
- semantic configuration;
- expected references;
- expected relationships.

---

# 16. Verify in the Direction of the Mutation

If installation copies:

    LOCAL → CONTAINER

then verification should compare:

    expected LOCAL state
    against
    actual CONTAINER state.

If rollback copies:

    BACKUP → CONTAINER

then verification should compare:

    BACKUP state
    against
    actual CONTAINER state.

Do not verify against an intermediate or regenerated source that could itself
have changed.

The verification source must correspond to the state the operation intended
to restore.

---

# 17. Verify More Than the Command Exit Code

For external commands such as:

- `docker`;
- `curl`;
- `cp`;
- `mv`;
- `rm`;
- `tar`;
- `sed`;
- `grep`;
- `awk`;
- `jq`;

ask:

    "What does exit 0 actually guarantee?"

Then determine whether additional verification is required.

External commands are dependencies.

Treat their exit status as evidence, not proof of the final system state.

---

# 18. Detect Silent Corruption

If the system already recognizes a possibility such as:

    command exits 0
    but output is corrupt

then apply the same reasoning to recovery.

Do not build asymmetric reliability:

    installation verifies integrity
    rollback assumes integrity.

Where integrity matters, use:

- SHA256;
- byte comparison;
- semantic validation;
- API GET-back verification;
- size checks;
- schema validation.

Apply the same standard in both forward and recovery paths when the failure
mode is relevant in both directions.

## 18a. Verification Itself Can Have Side Effects

Do not assume a verification step is a passive observation.

Examples of verification actions that are not actually side-effect-free:

- a `docker exec` health check that also rotates a log file or consumes a
  one-time token;
- a verification `GET` that increments a request counter used elsewhere
  for rate limiting or billing;
- a `curl` verification call that triggers a webhook on the receiving
  service;
- re-reading a queue or a one-shot notification channel as "verification"
  when the read itself consumes the message.

If a verification action mutates state, it is not purely verification: it
is a second mutation wearing a verification label. Review verification
code with the same "what does this actually change?" question applied to
every other mutation in this skill, and if a side effect is unavoidable,
document it explicitly rather than treating the check as free.

---

