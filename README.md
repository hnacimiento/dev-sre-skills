# dev-sre-skills

Nine [Claude Skills](https://www.anthropic.com/news/skills) that apply Site
Reliability Engineering reasoning to software development, operations, and
AI-assisted automation. They cover shell scripting, security, observability,
release/deployment, incident review, documentation, testing, service level
objectives, and an overarching engineering mindset that ties the rest
together.

## ⚠️ Status: partially validated, not exhaustively tested

**These skills have been tested against real, live agent sessions performing
real tasks — but only lightly, not exhaustively.** Each of the 9 skills was
run through 3 realistic test cases, each case executed once with the skill
available and once without it (a baseline), then graded against objective,
pre-defined assertions and independently reviewed. In every one of these 24
cases, the skill-available run outperformed the baseline, and the routing
to the right `references/*.md` file was confirmed correct — including one
case where the correct behavior was to open zero reference files. A
second, skeptical read of all 48 responses against the actual skill
content found four concrete issues, all of which were fixed.

That said: 3 cases per skill is a small sample, not exhaustive coverage. It
does not mean every section of every skill has been exercised, or that the
skills behave correctly on every realistic task, edge case, or adversarial
input. Earlier validation rounds, before this one, were self-graded — the
same model that wrote a skill also simulated and graded the response,
which is useful for catching internal contradictions but not the same as
the empirical rounds described above. Treat this repository as
tested-but-not-hardened, not as a proven system with exhaustive coverage.

## Install as a plugin

This repository is packaged as an installable Claude Code plugin (and its
own marketplace catalog). Instead of copying the 9 skill folders by hand
into each project's `.claude/skills/`, install once:

```
/plugin marketplace add hnacimiento/dev-sre-skills
/plugin install dev-sre-skills@dev-sre-skills
```

When Claude Code asks for an install scope, choose **User scope** to make
the 9 skills available in every project on your machine automatically, or
**Project**/**Local scope** to limit it to one repository — see
[`docs/PLUGIN_USAGE.md`](docs/PLUGIN_USAGE.md) for a full, beginner-friendly
walkthrough of installation, scope choice, and day-to-day use, including a
dedicated section on choosing between "every project" and "one project in
particular."

Installing the plugin does not change what the skills do or how they
reason — same `SKILL.md` files, same `references/`. It only changes how
they reach a project.

## How these skills activate

All 9 skills are **model-invoked**: Claude Code decides on its own, from
each skill's `description`, whether a given task calls for it — there is
no command you type to turn one on. Ask for a review of a bash deployment
script and `sre-bash` and `sre-engineering-mindset` activate on their own;
ask about SLO burn-rate alerting and `sre-slo` activates. You can still
invoke one explicitly if you want to (`/dev-sre-skills:sre-security`, once
installed as a plugin), but that is the exception, not the normal way of
using them.

## What's in here

Each skill lives under `skills/<name>/` (the layout the plugin format
requires).

| Skill | Purpose |
|---|---|
| `sre-engineering-mindset` | The parent reasoning framework: control loops, blast radius, toil, the Hybridization Rule, STPA, and a six-question cognitive router that points into the other eight skills. |
| `sre-bash` | Bash-specific reliability: exit codes, traps, idempotency, concurrency, recovery, generated scripts. |
| `sre-security` | Threat modeling, secrets, least privilege, the three-rung access ladder, and the Revert-First vs. Investigate-First incident hand-off. |
| `sre-observability` | Evidence over assertion: telemetry design, verification, agentic observability. |
| `sre-release-deployment` | Change as a reliability event: rollout state machines, rollback, canaries, the Checklist Trap. |
| `sre-incident-review` | Blameless-not-causeless postmortems, evidence discipline, causal analysis, the AI learning loop. |
| `sre-documentation` | Documentation as an operational contract — claims need evidence, not aspiration. |
| `sre-testing` | Failure-first and postcondition testing, fault injection, testing AI-assisted operations. |
| `sre-slo` | Error budgets, multi-window burn-rate alerting, freeze policy — with an explicit caveat against treating industry numbers as universal targets. |

## Architecture: lean core + load-on-demand references

Seven of the nine skills follow the same packaging pattern: each `SKILL.md`
stays small — frontmatter, purpose, the core mental model, and a
load-on-demand index — while the deeper material lives in that skill's
`references/*.md` files, opened only when the situation actually calls for
them. This mirrors the calibration the skills themselves teach
(`sre-engineering-mindset` §1a): apply machinery proportional to blast
radius, not to how much material exists.

`sre-testing` and `sre-slo` were deliberately left as single files — at
21–24KB they were already smaller than a typical reference file in the
other seven skills, and splitting them further would have been the same
disproportionate-machinery mistake the skills warn against.

## The three non-negotiable directives

Underneath the nine skills sits a short manifesto with three directives that
every skill was checked against:

1. **Hybridization Rule** — probabilistic reasoning (human judgment, an LLM
   agent's interpretation) is for adapting to ambiguity; deterministic
   mechanisms are required wherever the system needs a verifiable
   guarantee.
2. **Risk Asymmetry** — destructive/mutating actions get proportionally
   stronger controls (Zero Touch Production, Multi-Party Authorization)
   than constructive ones; access tier scales with blast radius, not by
   rote escalation.
3. **Postmortem Nutrition Loop** — every consequential incident must feed
   back into documentation, tests, and evaluation data, so neither the
   human operator nor a participating agent atrophies or drifts.

## License

MIT — see `LICENSE`.
