---
name: sre-bash
description: >
  SRE-focused Bash engineering skill for designing, reviewing, debugging,
  testing, and operating production Bash scripts. Apply when Bash is used for
  automation, deployment, installation, recovery, maintenance, orchestration,
  infrastructure operations, or other consequential production workflows.
  Focus on Bash's real execution semantics, explicit operational state,
  failure containment, verification, recovery, concurrency, security,
  observability, and safe interaction with external commands.
---

# SRE Bash Engineering

## 0. Purpose

Bash is a programming language, process orchestrator, and interface to external
systems.

It is powerful enough to automate production.

It is also permissive enough to create misleading states, partially completed
operations, silent data loss, unsafe recovery, and false success.

The purpose of this skill is to apply SRE reasoning specifically to Bash.

The objective is not:

    "write idiomatic Bash."

The objective is:

    "build a Bash program whose operational state remains understandable,
     bounded, observable, recoverable, and truthful when reality differs
     from the happy path."

Bash-specific reliability must account for:

- shell execution semantics;
- subprocesses;
- pipelines;
- exit statuses;
- traps;
- signals;
- `set -e`;
- `set -u`;
- pipelines and `pipefail`;
- command substitutions;
- subshells;
- functions;
- `source`;
- external commands;
- temporary files;
- filesystem state;
- concurrency;
- generated scripts;
- environment differences;
- partial mutation;
- interrupted execution.

---

# 1. Bash Is Not the System

A Bash script is usually a controller sitting between multiple systems:

    Bash
      ↓
    filesystem
      ↓
    operating system
      ↓
    external commands
      ↓
    containers / APIs / services
      ↓
    persistent production state

Bash itself may execute correctly while the resulting system state is wrong.

Therefore:

    shell correctness != operational correctness

and:

    exit 0 != desired state verified.

Always reason about the external state the script is trying to create.

---

## 1a. Where the Rest of This Reasoning Lives (Load-on-Demand Index)

Calibrate first with `sre-engineering-mindset` §1a — blast radius, not
script length, decides how much of this skill's machinery is load-bearing.
Everything past this section now lives in five reference files under this
skill's `references/` folder, opened only when the calibration says they
matter for the script in front of you.

- **`references/contracts-and-recovery.md`** (§2–§18a) — the operational
  contract, `set -Eeuo pipefail` and `set -e`/`set -u` scrutiny, error
  suppression, exit codes as an operational API, explicit state before
  mutation, the mutation boundary, backup before mutation, recovery as its
  own Bash program, per-resource and aggregate recovery honesty, failure
  containment during recovery, postconditions, and verifying more than the
  exit code (including detecting silent corruption and verification's own
  side effects). Open this for any script that mutates state that must
  survive the script's own failure.
- **`references/concurrency-and-lifecycle.md`** (§19–§30) — idempotency,
  temporary files, atomic file updates and their limits, concurrency and
  lock scope, signals, trap reentrancy, avoiding recursive recovery,
  `SIGKILL`, cleanup versus recovery, and `source` as executable input.
  Open this whenever the script could run concurrently with itself or
  must survive being interrupted mid-mutation.
- **`references/secrets-and-generated-code.md`** (§31–§47) — secrets,
  quoting, arrays, `eval` as a red flag, pipelines, command substitution,
  subshells, `read`, structured data, external commands as untrusted
  dependencies, container operations, and the parity/validation/
  self-containment requirements for generated Bash and generated recovery
  artifacts. Open this whenever the script handles secrets, untrusted
  input, or is itself machine-generated.
- **`references/preconditions-logging-and-state.md`** (§48–§65) — file
  deletion, dangerous empty values, preconditions before mutation, not
  overtrusting preflight, TOCTOU, logging, actionable error messages,
  preserving the primary failure, retry discipline, network operations,
  environment/portability, dependency detection, and the distinction
  between lock files, shared state, and durable state. Open this for
  anything that reads environment or network state before deciding to
  mutate.
- **`references/testing-and-final-review.md`** (§66–§78) — testing
  strategy, fault injection, testing the negative space, ShellCheck's
  limits, syntax validation, never trusting generated code unexecuted,
  operational contracts for functions, hidden side effects, when a script
  has outgrown Bash, code review questions, the Bash reliability
  invariants, and the final Bash review. Open this before considering a
  script finished or reviewed.

A script that scores low on `sre-engineering-mindset` §1a's signals still
owes itself the three-item floor from that skill — it does not need all
five reference files read end to end.

---
