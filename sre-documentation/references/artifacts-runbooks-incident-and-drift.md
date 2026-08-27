# sre-documentation — Reference: Artifacts, Limitations, Runbooks, Incident Docs, and Drift (sections 30-37)

This file is loaded on demand from `sre-documentation/SKILL.md`'s
load-on-demand index, when documenting generated artifacts, limitations, runbooks, incident writeups, or checking for documentation drift. It is not a standalone skill — it assumes
sre-engineering-mindset §1a has already calibrated the situation and
sre-documentation's own core has already been read.

---

# Artifacts

## 30. Document Generated Artifacts

If the system creates:

- backups
- manifests
- rollback scripts
- checksums
- logs
- state files
- reports
- temporary files
- generated configuration
- agent traces

document:

- location
- purpose
- creation conditions
- ownership
- permissions
- retention
- deletion behavior
- security sensitivity
- recovery use

An artifact used for recovery is part of the recovery architecture.

---

# Known Limitations

## 31. Limitations Are Part of the Contract

Document relevant limitations such as:

- SIGKILL prevents traps from running
- power loss prevents cleanup
- external APIs may be unavailable
- application health may not be automatically verified
- eventual consistency may delay verification
- upstream artifacts may change
- recovery may require credentials
- backups may not contain all state
- rollback may not reverse external side effects
- dry-run may differ from real execution
- agent reasoning may be nondeterministic
- human approval may be unavailable
- monitoring may not detect every failure

Do not hide limitations because they make the system look less polished.

Accurate limitations improve operational safety.

---

# Runbooks

## 32. Runbooks Must Be Executable

A runbook should be usable by someone who did not write the system.

Prefer:

    SYMPTOM
        ↓
    DIAGNOSIS
        ↓
    SAFETY CHECK
        ↓
    ACTION
        ↓
    EXPECTED RESULT
        ↓
    FAILURE INTERPRETATION
        ↓
    RECOVERY
        ↓
    ESCALATION

Each action should specify:

- prerequisites
- expected output
- interpretation
- safety conditions
- recovery if it fails

Avoid instructions that require guessing.

---

## 33. Runbooks Must Preserve Decision Context

Do not only document commands.

Explain:

- why the command is used
- what signal justifies it
- what state it assumes
- what state it changes
- when not to use it
- what result indicates success
- what result requires escalation

This prevents cargo-cult operations.

---

# Incident Documentation

## 34. Preserve Operational Reality

Incident documentation should preserve:

- impact
- evidence
- timeline
- detection
- hypotheses
- decisions
- actions
- recovery
- contributing conditions
- failed controls
- successful controls
- unknowns
- corrective actions
- validation

Do not rewrite the incident into an artificially clean narrative.

Production work may be:

- concurrent
- uncertain
- adaptive
- contradictory
- incomplete
- improvised

Document that reality.

---

## 35. Work-as-Imagined vs Work-as-Done

When documenting operational behavior compare:

### WORK-AS-IMAGINED

What documentation, architecture, or procedures expected.

### WORK-AS-DONE

What operators actually did.

Look for differences in:

- commands
- procedures
- tools
- permissions
- communication
- timing
- workarounds
- coordination
- decision-making

Do not automatically classify deviations as violations.

Ask:

    What property of the operational environment made this adaptation reasonable?

---

# Documentation Drift

## 36. Documentation Is a Regression Surface

Documentation can become stale even when the implementation remains correct.

Look for:

- commands that no longer work
- renamed configuration
- changed paths
- obsolete screenshots
- stale examples
- incorrect exit codes
- missing failure states
- obsolete architecture
- outdated permissions
- incorrect recovery instructions
- claims no longer supported by tests

Important documentation should be reviewed when behavior changes.

---

## 37. Detect Documentation Drift

When modifying a system, ask:

    What documentation claims does this change invalidate?

Check:

- README
- architecture documents
- runbooks
- recovery procedures
- security documentation
- examples
- generated documentation
- incident procedures

Do not update documentation only when explicitly requested if the requested implementation change clearly invalidates an operational claim.

---

