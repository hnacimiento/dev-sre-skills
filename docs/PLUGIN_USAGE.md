# How to use the `dev-sre-skills` plugin

This guide assumes you have never used a Claude Code plugin before. You do
not need to understand the plugin system's internals — following the steps
below is enough.

## What this is

`dev-sre-skills` is an installable package containing the 9 skills in this
repository (`sre-engineering-mindset`, `sre-bash`, `sre-security`,
`sre-observability`, `sre-release-deployment`, `sre-incident-review`,
`sre-documentation`, `sre-testing`, `sre-slo`). Previously, using them in a
project meant copying the folders by hand. Now they install with two
commands and become available wherever you choose — in every project, or
in just one.

**Important:** installing the plugin does not change what the skills do or
how they reason in any way. They remain exactly the same `SKILL.md` and
`references/` files as always. The only thing that changes is *how they
reach a project*.

## Step 1: Install Claude Code (if you don't already have it)

If you already use Claude Code on your machine, skip to Step 2.

If not, follow the official installation instructions at
https://code.claude.com — the installer varies by operating system (on
Windows there is a native installer, or you can use `npm`).

## Step 2: Add this repository as a "marketplace"

Open a terminal where you have Claude Code (`claude`) and, inside a Claude
Code session, run:

```
/plugin marketplace add hnacimiento/dev-sre-skills
```

This does not install anything yet — it just tells Claude Code "this
GitHub repository has plugins available." You do this **once per
machine**, not once per project.

## Step 3: Install the plugin

With the marketplace added, run:

```
/plugin install dev-sre-skills@dev-sre-skills
```

Claude Code will ask you which **scope** to install it at. This is where
you decide whether you want the skills in every project or in one
particular project — see the section below.

## All projects, or one in particular?

When the installer asks for a scope, you have three options. This is the
question most people actually care about, so here it is in full, plain
terms:

### Option A — Every project on this machine (recommended default)

Choose **User scope**. The plugin installs at your user level on this
machine, and from that point on it is automatically available in every
project you open with Claude Code — nothing further to do, no per-project
setup. This is the equivalent of what you used to get by copying the
folders into `~/.claude/skills/`, except it now happens once and updates
centrally.

Use this when: you want these SRE skills available by default everywhere
you work, which is the typical case for your own skill set.

### Option B — Every collaborator on one specific repository

Choose **Project scope**. The plugin gets declared in that repository's
own configuration (`.claude/settings.json`), so anyone who clones that
project and trusts the folder gets the plugin too, without installing it
themselves.

Use this when: you are working on a shared repository with other people
and want everyone on that project to have the same skills available, but
you don't necessarily want it everywhere else on your machine.

### Option C — Just you, just this one repository

Choose **Local scope**. The plugin is available only to you, only inside
this one repository — it does not follow you to other projects and it is
not shared with collaborators.

Use this when: you want to try the plugin on one project first, before
deciding whether to install it everywhere (Option A).

### Turning it off for one project without uninstalling everywhere

If you installed with **User scope** (Option A, all projects) and later
want a specific project to *not* use it, you don't need to uninstall it
globally. From inside that project, run:

```
/plugin disable dev-sre-skills@dev-sre-skills
```

This disables it only for that one project; it stays active everywhere
else.

## Step 4: Confirm it's active

After installing, Claude Code will tell you whether the plugin is already
active ("Plugin is now active") or whether you need to run:

```
/reload-plugins
```

to activate it without restarting the terminal.

To check what you have installed at any time:

```
/plugin list
```

## Day-to-day use

There is nothing else to learn for normal use: the skills still trigger on
their own, exactly as before, whenever Claude Code detects that the task
you asked for matches what a skill describes (for example, if you ask it
to review a bash script bound for production, `sre-bash` and
`sre-engineering-mindset` activate on their own). You do not need to
invoke them by name.

If you ever want to force the use of a specific skill instead of letting
Claude decide on its own, its name carries the plugin's prefix — for
example `/dev-sre-skills:sre-security` instead of just referencing it —
but this is optional and not needed for normal use.

## When the skill content gets updated

Whenever something in this repository is corrected or improved and pushed
to GitHub, bring that update into your installation from any Claude Code
session with:

```
/plugin marketplace update dev-sre-skills
```

followed by `/reload-plugins` if needed. This replaces having to go
project by project rewriting files by hand.

## If something isn't working

- If `/plugin` says the command doesn't exist: update Claude Code to the
  latest version (`npm install -g @anthropic-ai/claude-code@latest` if you
  installed it via npm, or the matching native installer) and restart your
  terminal.
- If you installed the plugin but don't see the skills triggering: run
  `/plugin list` to confirm it shows as installed and enabled, and run
  `/reload-plugins` if needed.
- As a last resort, if the problem persists: `rm -rf
  ~/.claude/plugins/cache` (this clears the plugin system's local cache,
  not your repository) and reinstall.
