---
name: authoring-skills
description: Use when creating a new skill for the AI Activity model — a `_skills/<skill-name>/SKILL.md` file under any activity template directory. Covers where the file lives, the YAML frontmatter contract (name/description) and how the description drives on-demand loading, how skills flow from `<activity>/_skills/` to `<project>/.claude/skills/` via a single directory symlink at `initiate` time, the fallback to `mother/_skills/`, the sub-install exception (skills are inherited, not installed), naming conventions, content guidance (imperative task-focused, one skill per procedure), and when to extract from AI-Activity.org versus keeping content always-loaded.
---

# Authoring an AI-Activity Skill

A **skill** is an on-demand knowledge pack that Claude Code loads
only when its `description` matches the task at hand. Skills live
under an activity template directory and are surfaced to Claude via
the `<project>/.claude/skills/` symlink that `startAiActivity.cs -i
initiate` installs.

Read the AI-Activity.org that ships with the `aiActivity` template
first if you are not already familiar with the AI Activity model —
this skill assumes those concepts.

## Where the File Goes

Canonical path in the templates repo:

```
<templatesBase>/<activity>/_skills/<skill-name>/SKILL.md
```

Notes:

- **The underscore prefix `_skills/` is deliberate.** Skills are
  activity content, kept at the activity root so they are
  independently visible to humans and other tools — not buried
  under `_claude/`. The engine bridges the two by symlinking
  `_skills/` into `.claude/skills/` at install time.
- **One directory per skill, containing `SKILL.md`.** Additional
  files (referenced examples, sample scripts) may live alongside
  `SKILL.md` in the same directory. Claude Code loads `SKILL.md`;
  other files are available but not auto-loaded.
- **The skill's directory name is what appears at
  `.claude/skills/<skill-name>/`.** Choose it carefully — it is
  effectively the skill's public name.

## Frontmatter Contract

Every `SKILL.md` begins with YAML frontmatter:

```yaml
---
name: <skill-name>
description: <one paragraph — see below>
---
```

Two required fields. Anything else is currently ignored.

### `name`

Kebab-case. Should match the enclosing directory name. Verb- or
noun-oriented depending on what reads best:

- `writing-cs-commands` (verb + object — the task)
- `dblock-authoring` (object + activity)
- `readme-authorship`, `bisos-pip-release` (activity)

Prefer specific over generic: `bisos-pip-release`, not `release`.

### `description`

**This is what Claude uses to decide whether to load the skill.**
Not a title. Not a summary of the skill's structure. It should read
as an answer to "when should you load this?" — describing the
concrete situations, artifacts, or symptoms that indicate a match.

Guidelines:

- Start with `Use when …` or an equivalent trigger phrase.
- Name the specific artifacts, file patterns, or command shapes
  involved (`.cs` files, `pyproject.toml`, `pypiProc.sh`, a
  `####+BEGIN:` line, a Blee panel's `_nodeBase_/` directory).
- Enumerate multiple triggering conditions when they exist —
  the description is prose, not a fixed schema, so use commas or
  "either / or" phrasing freely.
- Long is fine. Sparse and generic descriptions cause misses and
  false loads.
- Do not describe the skill's own /structure/ ("this skill has three
  sections…"). Describe /the situation/ that should trigger it.

Compare — weak:

```yaml
description: Guide for writing bisos-pip release notes.
```

Strong:

```yaml
description: Use when preparing a release of a bisos-pip package — cutting a version bump in setup.py or pyproject.toml, running `pypiProc.sh -i fullPrep`, uploading to PyPI, tagging the git commit, or writing the release changelog entry. Covers version-bump conventions, the fullPrep pipeline, and the tag-and-push sequence.
```

The strong version names artifacts (`setup.py`, `pypiProc.sh`) and
tasks (cutting a bump, uploading, tagging) so it fires when the
user says "let's release bisos.foo" without them explicitly naming
the skill.

## Content Structure (After the Frontmatter)

The body is Markdown. Some patterns that work well:

### Lead with the shape / decision tree

Open with the smallest thing the reader needs to know to orient — a
diagram, a decision tree, a "what are the three cases" table. Do
not open with prose backstory.

### Imperative headings, task-focused

Prefer `## Writing the Body`, `## Choosing a Version Number`, `##
When to Use classHead vs classHeadNoTracked` over `## Overview` and
`## Introduction`. Each section should answer a concrete question a
reader will have at that step.

### Show, don't summarize

Code snippets, file excerpts, and command invocations carry more
information per line than prose. Use them.

### Cross-reference cautiously

A skill is loaded on demand, so a reader may not have loaded the
surrounding context. If you reference another skill, name it
explicitly ("load the `dblock-authoring` skill first"), not just
"see the related skill". If you reference project files, use paths
relative to a stable root or use placeholders (`<projectRoot>`,
`<templatesBase>`).

### Length

There is no hard limit. Skills of 100–800 lines are typical. If a
skill grows past ~1000 lines, ask whether it is really two skills.

## How Skills Flow to `.claude/skills/`

At `startAiActivity.cs -i initiate` time:

1. The engine looks for `<templatesBase>/<activity>/_skills/`.
2. If that directory exists, it creates a symlink at
   `<project>/.claude/skills/` pointing to it.
3. If not, it falls back to `<templatesBase>/mother/_skills/`.
4. If neither exists, no symlink is created (Claude Code simply
   sees no skills for this activity).

The symlink is a **single directory-level symlink**, not one per
skill. That means:

- Adding a new skill under `<activity>/_skills/<new-skill>/` shows
  up immediately in every project already `initiate`-d against
  that templates base — no re-initiate needed. The live symlink
  is the propagation mechanism.
- Renaming or removing a skill in the templates repo removes it
  from every installed project on the next Claude session.
- Individual skills cannot be shadowed per-project through the
  standard install path. If a project needs a project-specific
  skill, add it to the project's `.claude/skills/` by hand after
  `initiate` (a per-project skill is currently outside the model —
  the templates repo is the single source of truth).

## The Sub-Install Exception

`startAiActivity.cs -i initiateSub` does **not** install
`.claude/skills/`. Sub installs are slim: they get only the
per-effort files (`CLAUDE.md`, `AI-Activity.org`, `AI-DevStatus.org`,
`AI-WorkPlan.org`). Skills are inherited from the ancestor's
`.claude/` via Claude Code's walk-up.

Implication for authoring: **a skill's audience is any project of
this activity, whether installed at the repo top or as a sub.**
Don't assume the reader is at the repo root. If a skill references
project-relative paths, remember that in a sub install the "project
root" that matters may be one or more directories up.

## When to Write a Skill vs Adding to AI-Activity.org

`AI-Activity.org` is **always loaded** by every Claude session for
projects of this activity. Skills load **only on demand**. The
choice between them is a bandwidth choice.

Put content in `AI-Activity.org` when it is:

- Applicable to every session, regardless of task ("this project
  uses type hints for new code").
- Short and orienting (the "what kind of work is this" material).
- Necessary for the reader to make basic decisions before
  attempting anything.

Extract content into a skill when it is:

- Procedural — a specific how-to for a specific task.
- Only relevant for some fraction of sessions.
- Long enough that always-loading it costs more than it earns.
- Discoverable by a discriminating `description` (i.e., you can
  write a plausible "use when …" phrase for it).

The `bisos-pip` activity's history illustrates the split: an
original 456-line `AI-Activity.org` was reduced to 154 lines of
always-loaded orientation, with ~916 lines of procedural content
moved into five skills (`writing-cs-commands`, `dblock-authoring`,
`readme-authorship`, `blee-panel-authorship`, `bisos-pip-release`).
Each of those triggers only when the reader is doing the specific
kind of work it addresses.

## Naming Conventions Recap

- Directory and `name:` field: kebab-case, identical.
- Verb-forward for task skills (`authoring-skills`, `writing-cs-commands`).
- Noun-forward for artifact skills (`dblock-authoring`,
  `bisos-pip-release`).
- Prefer specific over generic. `bisos-pip-release` beats
  `release`; `blee-panel-authorship` beats `panels`.
- Do not prefix with the activity name — the enclosing
  `<activity>/_skills/` directory already provides that context.
  `writing-cs-commands` under `bisos-pip/_skills/`, not
  `bisos-pip-writing-cs-commands/`.

## Testing a New Skill

After writing the SKILL.md:

1. Confirm it appears at `<project>/.claude/skills/<skill-name>/SKILL.md`
   via the symlink — a quick `ls -L .claude/skills/` in a project
   `initiate`-d against your templates base should show it.
2. In a fresh Claude Code session, describe a task that the
   `description` should trigger on. If Claude does not load the
   skill, the `description` is not discriminating enough — rewrite
   it with more concrete artifacts and task language.
3. Describe a task that should /not/ trigger it. If Claude loads
   the skill anyway, the `description` is too broad.

Iteration on the `description` is normal. The body of the skill
usually stabilizes faster than the description.

## Common Pitfalls

- **Frontmatter YAML syntax errors** silently break the skill.
  Keep it simple: two fields, quoted only if you must include a
  `:` or leading whitespace-sensitive content.
- **Describing the skill instead of the trigger.** "This skill
  covers X" is not a description; it's a title. Rewrite as "Use
  when doing X".
- **Splitting a skill across two SKILL.md files** for length
  reasons. Long is fine; two overlapping skills confuse the
  loader. If the content really is two topics, split by topic,
  not by length.
- **Putting always-needed context in a skill.** If the reader
  cannot make basic decisions without it, it belongs in
  `AI-Activity.org`.
- **Using project-specific paths in a skill that ships with a
  reusable activity.** Prefer `<projectRoot>`, `<templatesBase>`,
  `<activity>` placeholders.
