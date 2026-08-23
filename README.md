# SDD Skill

The version-controlled source for the `spec-driven-development` Claude
Skill — the process knowledge (not any one project's content) that
governs how spec-driven development gets run with Claude, Claude Code,
and Claude Design. New to spec-driven development, or wondering why
you'd want it? [`SDD-Template`](https://github.com/EHaake/SDD-Template)'s
README has the full overview — this one assumes that's already decided
and covers getting the skill itself installed and kept current.

**This repo is not itself read by Claude Code or claude.ai.** It's the
place changes get made and history gets kept; the skill only actually
functions once its contents are installed to the two places below.
Think of this repo as the source of truth you edit and periodically
sync outward, not a live location either tool reads from directly.

## What's here

```
SKILL.md                        AI-facing instructions — the file that
                                 actually gets loaded once installed
assets/
  CLAUDE-template.md            Constitution skeleton
  spec-template.md              spec.md skeleton
  plan-template.md              plan.md skeleton
  tasks-template.md             tasks.md skeleton
  design-brief-template.md      brief.md skeleton, for projects with a UI
  skeptical-reviewer.md         Source copy of the reviewer subagent —
                                 see "Installing" below for where the
                                 active copy actually lives
references/
  collaboration-workflow.md     Full step-by-step version of the
                                 routine/subagent/escalate triage
```

`README.md` (this file) and `How-To-Use.md`, if you keep a copy here,
are for humans — nothing in this repo besides `SKILL.md`, `assets/`, and
`references/` is read by either tool.

## Installing

Two separate installs, two separate places — updating this repo doesn't
propagate to either automatically.

### Claude Code

Skills are only discovered from an exact location:
`~/.claude/skills/spec-driven-development/` (personal, every project on
this machine) or a project-level `.claude/skills/spec-driven-development/`
(that one repo only). Copy `SKILL.md`, `assets/`, and `references/` there
directly — same folder structure as this repo, just at that path instead.

Verify it's actually recognized, not just present: open Claude Code
anywhere and ask what skills are available.

### claude.ai

Settings → Capabilities → enable "Code execution and file creation"
(skills won't appear in the menu until this is on). Then zip `SKILL.md`,
`assets/`, and `references/` together — the zip's root should be the
`spec-driven-development` folder itself, not the loose files and not an
extra wrapper folder around it. Settings → Customize → Skills → upload
the zip, toggle it on.

Verify with a fresh chat: ask it to start a new SDD project and confirm
it references the methodology unprompted.

## Updating

Edit here first, commit normally. Then manually re-sync both install
locations — copy the changed files to the Claude Code path, re-zip and
re-upload for claude.ai. Nothing pushes automatically to either; this
repo having the fix doesn't mean either installed copy has it yet.

## The companion repo

**[`SDD-Template`](https://github.com/EHaake/SDD-Template)** is the
project-scaffold counterpart — a GitHub template
repo (the actual "Template repository" feature) that seeds a new
project's `CLAUDE.md`/`spec.md`/`plan.md`/`tasks.md`/`brief.md`. This
repo and that one are deliberately separate: this one is the
methodology, installed once, applying to every project; that one is a
starting point, cloned fresh per project, with no ongoing link back to
either this repo or itself once used.
