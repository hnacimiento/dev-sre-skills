# dev-sre-skills

Nine [Claude Skills](https://www.anthropic.com/news/skills) that apply Site
Reliability Engineering reasoning to software development, operations, and
AI-assisted automation. They cover shell scripting, security, observability,
release/deployment, incident review, documentation, testing, service level
objectives, and an overarching engineering mindset that ties the rest
together.

## ⚠️ Status: not empirically validated

**These skills have not been tested against a real, live agent session
performing a real task.** Everything validated so far was a self-graded
adversarial battery: the same model that wrote a skill also wrote the test
prompts, simulated how an agent would respond to them, and graded that
simulated response against its own criteria. That is useful for catching
internal contradictions and missing cases, but it is not the same as
empirical evidence that the skills behave correctly in actual use — whether
an agent picks the right skill, actually follows the load-on-demand
references at the right moment, or reasons correctly under a real,
adversarial, or ambiguous task.

Treat everything in this repository as a well-reasoned draft, not a proven
system. `BATERIA_ADVERSARIAL_RESULTADOS.md` and `REVISION_CRITICA_SKILLS_SRE.md`
document the full history of what was reviewed, what was found, and what
remains open — including this caveat.

## What's in here

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
