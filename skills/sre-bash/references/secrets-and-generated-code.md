# sre-bash — Reference: Secrets, Quoting, and Generated Scripts (sections 31-47)

This file is loaded on demand from `sre-bash/SKILL.md` §1a, whenever secrets, untrusted input, or Bash-generating-Bash are involved. It is
not a standalone skill — it assumes sre-engineering-mindset §1a has
already calibrated the situation and sre-bash's own core has already
been read.

---

# 31. Secrets

Never expose secrets unnecessarily.

Avoid:

- command-line arguments containing secrets;
- `echo` of secrets;
- debug output containing secrets;
- generated scripts containing plaintext secrets;
- unnecessary environment propagation;
- passing secrets through visible process arguments.

When checking whether a secret leaked into an artifact:

    search for the actual secret value

not merely:

    search for the variable name.

Guard against empty secrets.

An empty search pattern can produce meaningless matches.

Security validation must verify the actual security property.

---

# 32. Do Not Create New Secret Exposure While Fixing Security

A security check can itself introduce exposure.

For example, passing a secret as an argument to:

    docker exec sh -c '...'

may create a new process-visible representation.

When fixing a secret-detection mechanism, ask:

    "Does the fix create a larger exposure surface than the original issue?"

Prefer designs where the secret remains in the narrowest possible trust domain.

---

# 33. Quoting

Quote variables by default:

    "$variable"

Unquoted expansion can introduce:

- word splitting;
- pathname expansion;
- unexpected argument boundaries.

Treat unquoted variables as intentional exceptions requiring a reason.

Particular care is required for:

- filenames;
- user input;
- URLs;
- paths containing spaces;
- wildcard characters;
- newlines.

---

# 34. Arrays

Use arrays for lists of arguments.

Prefer:

    command "${args[@]}"

over constructing command strings and evaluating them.

Avoid:

    eval

unless there is an extremely strong reason and the input is completely
controlled.

Do not construct shell code from untrusted data.

---

# 35. `eval` Is a Red Flag

Whenever `eval` appears, stop and ask:

- Why is it necessary?
- Can arrays solve the problem?
- Can direct invocation solve it?
- Can a function solve it?
- Is the input trusted?
- Can shell metacharacters enter it?

Dynamic shell evaluation creates both reliability and security risk.

---

# 36. Pipelines

Understand what pipeline status means.

Without `pipefail`, a pipeline may report the status of the last command even
if an earlier command failed.

With:

    set -o pipefail

pipeline failures become more visible.

But still ask:

    "What does the pipeline as a whole mean operationally?"

Do not treat `pipefail` as semantic verification.

For important pipelines, validate the resulting output.

---

# 37. Command Substitution

Command substitution:

    $(command)

can hide important distinctions.

Ask:

- What happens if the command fails?
- Is its output empty?
- Is empty output valid?
- Is whitespace significant?
- Is newline trimming acceptable?
- Does `errexit` behave as expected here?

Do not use command substitution as if it were a typed function returning a
guaranteed value.

Validate important results.

---

# 38. Subshells

Subshells can change state visibility.

For example:

    ( ... )

and:

    command | while read ...

may execute in a different shell context depending on the structure.

Do not assume variables modified in a subshell are available afterward.

For stateful loops, understand exactly where execution occurs.

---

# 39. `read`

When using `read`:

- handle EOF;
- handle whitespace intentionally;
- handle empty lines intentionally;
- consider backslashes;
- use appropriate flags;
- do not assume a newline always exists.

If parsing structured data, prefer a real parser.

Do not reinvent JSON parsing with fragile `grep` and `sed`.

---

# 40. Structured Data

Do not parse structured formats with text tools when semantics matter.

For JSON:

    use jq

when available.

For YAML:

    use an appropriate parser.

For machine-readable APIs:

    validate structure explicitly.

Text processing is appropriate for text.

It becomes dangerous when a text representation is mistaken for a formal data
model.

---

# 41. External Commands Are Untrusted Dependencies

Every command outside Bash can:

- disappear;
- change behavior;
- return unexpected output;
- return success incorrectly;
- be unavailable;
- behave differently by version;
- emit localization-dependent output.

Validate dependencies during preflight.

Prefer stable machine-readable interfaces.

Avoid parsing human-oriented output when a machine-readable API exists.

---

# 42. Docker and Container Operations

When Bash orchestrates Docker:

- verify the target container identity;
- do not rely solely on a cached container name;
- inspect actual container state;
- distinguish container absence from command failure;
- verify files after `docker cp`;
- understand container filesystem semantics;
- avoid leaking secrets through command arguments;
- verify postconditions inside the actual target.

A successful `docker cp` is not proof that the intended artifact is correct.

---

# 43. Generated Bash Scripts

Generated scripts are a special risk.

There are two execution times:

    generator time

and:

    generated-script runtime.

Variables may be evaluated at either time.

Heredocs may contain:

- escaped `$`;
- unescaped `$`;
- command substitutions;
- nested quotes;
- generated paths;
- generated secrets;
- generated functions.

Every generated script must be treated as independent code.

Do not assume:

    generator is valid

implies:

    generated script is valid.

---

# 44. Generated Script Parity

If a generated rollback/recovery script implements logic also present
in-process, treat them as two implementations of the same contract.

They must not silently diverge.

Test:

- syntax;
- shellcheck;
- normal execution;
- failure execution;
- recovery execution;
- exit status;
- postcondition verification.

A template bug is a runtime bug.

---

# 45. Validate Generated Artifacts Mechanically

Before relying on a generated Bash script:

    bash -n generated.sh

and, where appropriate:

    shellcheck generated.sh

Then execute it in controlled tests.

Do not assume that because the heredoc appears syntactically correct in the
generator, the generated file is correct.

Inspect generated output when debugging escaping issues.

---

# 46. Recovery Artifacts Must Be Self-Contained

If a recovery script may run much later:

- do not depend on the original Bash process;
- do not depend on in-memory variables;
- do not assume the original environment exists;
- do not assume secrets still exist;
- do not assume the original container identity remains valid.

Persist the information required for recovery.

If a required dependency is missing, recovery must report that explicitly.

It must never reinterpret:

    cannot recover

as:

    nothing needed recovery.

---

# 47. Verification of Generated Recovery

A generated rollback script should independently satisfy:

    preconditions
    →
    restore
    →
    verify
    →
    explicit result
    →
    explicit exit status

Do not assume the generator's own state can be consulted later.

The generated script must be operationally truthful on its own.

---

