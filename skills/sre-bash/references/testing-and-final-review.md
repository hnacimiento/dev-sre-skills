# sre-bash — Reference: Testing Strategy and Final Review (sections 66-78)

This file is loaded on demand from `sre-bash/SKILL.md` §1a, before treating any consequential script as finished. It is
not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-bash's own core has already
been read.

---

# 66. Testing Strategy

A production Bash script requires more than happy-path tests.

At minimum consider:

### Normal

- successful run;
- repeated run;
- expected input variants.

### Failure

- dependency missing;
- permission denied;
- external command failure;
- malformed input;
- missing resource;
- unexpected resource.

### Partial mutation

- failure after resource 1;
- failure after resource N;
- verification failure after successful command.

### Recovery

- successful rollback;
- partial rollback;
- failed rollback;
- missing backup;
- missing secret;
- unreadable secret;
- corrupted backup.

### Signals

- SIGINT;
- SIGTERM;
- interruption during mutation;
- interruption during recovery.

### Abrupt death

- SIGKILL;
- process termination during state write;
- process termination after partial mutation.

### Concurrency

- same target concurrently;
- different targets concurrently;
- shared state contention.

### Generated scripts

- syntax;
- ShellCheck;
- normal execution;
- failure execution;
- recovery execution.

---

# 67. Fault Injection

Do not merely test that a command returns non-zero.

Test realistic failure modes.

Examples:

    command returns non-zero

    command returns zero but output is corrupt

    API returns success but state is unchanged

    resource disappears after preflight

    filesystem becomes read-only

    secret file becomes unreadable

    generated script receives malformed state

    process receives SIGTERM during mutation

    verification temporarily fails

    container disappears during rollback

Fault injection should prove specific reliability properties.

---

# 68. Test the Negative Space

Ask:

    "What did the code assume would never happen?"

Then test it.

Examples:

- empty variable;
- missing file;
- duplicate entry;
- stale state;
- corrupted state;
- unexpected whitespace;
- unexpected filename;
- missing newline;
- API returning unexpected JSON;
- target renamed;
- target disappearing;
- concurrent execution.

The negative space often contains the real reliability bugs.

---

# 69. ShellCheck Is Necessary but Not Sufficient

Run ShellCheck.

Treat findings seriously.

But do not confuse:

    ShellCheck clean

with:

    operationally reliable.

ShellCheck can identify many shell-language hazards.

It cannot determine:

- whether rollback is correct;
- whether blast radius is acceptable;
- whether exit codes are truthful;
- whether postconditions are verified;
- whether a recovery contract is correct;
- whether two commands interact unsafely.

Use static analysis plus runtime testing plus SRE reasoning.

---

# 70. Bash Syntax Validation

Before executing a generated or modified script:

    bash -n script.sh

Syntax validation should be automatic where practical.

For generated scripts:

    generate
    →
    bash -n
    →
    shellcheck
    →
    controlled execution

Do not rely on visual inspection of heredocs.

---

# 71. Never Trust Generated Code Without Executing It

A generated script is production code.

It deserves:

- syntax validation;
- static analysis;
- behavioral tests;
- failure tests;
- recovery tests;
- security review.

The generator itself should have tests proving that generated output implements
the expected contract.

---

# 72. Bash Functions Need Operational Contracts

For important functions define:

- inputs;
- outputs;
- side effects;
- exit status;
- variables modified;
- files modified;
- external commands called;
- whether failure is recoverable;
- whether the function may trigger traps.

A function that returns zero should mean something explicit.

Avoid functions whose return code has no documented meaning.

---

# 73. Avoid Hidden Side Effects

Be cautious with functions that silently:

- modify global variables;
- change directories;
- change shell options;
- alter traps;
- mutate files;
- mutate external systems.

Make consequential side effects obvious.

This is especially important in large Bash scripts where control flow becomes
difficult to reason about.

---

# 74. Large Bash Scripts

As Bash grows, risk grows nonlinearly.

For large scripts:

- define phases;
- isolate responsibilities;
- minimize global state;
- centralize state transitions;
- centralize logging;
- centralize error classification;
- document mutation boundaries;
- keep recovery logic explicit;
- test functions independently where practical.

Do not let a 2,000-line Bash script become a collection of accidental control
flow interactions.

At some complexity threshold, evaluate whether Bash remains the right tool.

---

# 75. Know When Bash Has Become the Wrong Tool

Bash is appropriate for:

- orchestration;
- glue;
- filesystem operations;
- process execution;
- small administrative workflows;
- controlled deployment helpers.

Consider another language when the system requires substantial:

- structured data modeling;
- concurrency;
- complex state machines;
- long-lived processes;
- sophisticated testing;
- transactional behavior;
- API clients;
- error typing;
- complex parsing;
- complex domain logic.

Do not rewrite Bash merely because it is ugly.

Do not keep Bash merely because it already exists.

Choose based on operational risk and maintenance cost.

---

# 76. Code Review Questions

When reviewing production Bash, ask:

### Shell semantics

- Where can `set -e` behave differently than expected?
- Where is `|| true` hiding failure?
- Are pipelines correctly handled?
- Are subshell boundaries understood?
- Are command substitutions safe?

### State

- What state exists before mutation?
- What state exists after each mutation?
- How is partial state represented?

### Failure

- Can any failure produce false success?
- Can any failure skip final state aggregation?
- Can cleanup hide the primary failure?

### Recovery

- Does rollback itself have explicit state?
- Can recovery partially fail?
- Is recovery verified?

### Concurrency

- Can two executions overlap?
- Is shared state protected?

### Security

- Can secrets appear in arguments, logs, artifacts, or generated scripts?
- Does a security check actually test the security property?

### Verification

- Is success based on postconditions?
- Are silent corruption scenarios tested?

### Signals

- What happens on SIGINT?
- SIGTERM?
- SIGKILL?

### Generated code

- Has the generated script been syntax-checked?
- Has it been executed under failure conditions?

---

# 77. The Bash Reliability Invariants

For consequential Bash automation, prefer these invariants:

### B1 — Truthful success

The script must not report success unless the defined success postconditions
are verified.

### B2 — Explicit mutation state

The script must distinguish whether mutation never started, started, completed,
or partially completed.

### B3 — Recovery honesty

Recovery must report SUCCESS, PARTIAL, or FAILED after it begins.

### B4 — No silent omission

A resource requiring action must have an explicit outcome.

### B5 — Postcondition verification

Command exit status alone is insufficient when the resulting state matters.

### B6 — Failure containment

One resource failure should not prevent safe evaluation of other resources.

### B7 — Primary/recovery separation

Recovery success never converts a failed primary operation into success.

### B8 — Durable recovery

If recovery must survive process death, required recovery state must exist
outside process memory before mutation begins.

### B9 — Concurrency safety

Shared mutable state must have an explicit coordination strategy.

### B10 — Security property validation

Security assertions must verify the actual security property, not a proxy.

### B11 — Generated-code parity

Generated operational scripts must independently satisfy their intended
contract.

### B12 — Operator truth

The final output must allow an operator to understand what happened and what
remains unsafe.

---

# 78. Final Bash Review

Before declaring a consequential Bash script production-ready, answer:

    What exactly does exit 0 mean?

    Can exit 0 happen while the system is wrong?

    What happens if command #7 fails?

    What happens if command #7 returns 0 but produces bad state?

    What happens if SIGTERM arrives between commands #7 and #8?

    What happens if SIGKILL arrives there?

    What happens if the script runs twice?

    What happens if two copies run simultaneously?

    What happens if the target disappears?

    What happens if a required secret disappears?

    What happens if recovery fails?

    What happens if recovery partially succeeds?

    How is that partial state reported?

    How is recovery verified?

    What state survives process death?

    Can the operator reconstruct what happened?

    Can an attacker exploit the same failure path?

    Are generated scripts independently valid?

    Are all important assumptions tested?

    Is Bash still the appropriate implementation technology?

The final question is:

    "If this script fails in a way we did not anticipate,
     will it tell the truth about the resulting system state
     and leave an operator with a credible path to recovery?"

