# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Skills is a plugin collection of 10 reusable Claude Code skills for software engineering workflows. Skills are installed as a Claude Code plugin and provide structured, repeatable processes for tasks like implementation, code review, ticket management, and infrastructure investigation. The skills are tech-stack agnostic — they adapt to each project via its CLAUDE.md file.

## Repository Structure

```
.claude-plugin/plugin.json          # Root plugin metadata (aggregates all skills)
skills/<skill-name>/
  .claude-plugin/plugin.json        # Per-skill plugin metadata (for individual installation)
  skills/<skill-name>/
    SKILL.md                        # Skill definition (prompt, phases, rules)
    *.md                            # Supporting files (checklists, templates)
docs/                               # Requirements reference, customization guide
examples/                           # Example CLAUDE.md files for .NET, Node, Python stacks
```

Each skill follows the same directory convention: `skills/<name>/.claude-plugin/plugin.json` points to `skills/<name>/skills/<name>/SKILL.md`. The double nesting (`skills/<name>/skills/<name>/`) is required by the Claude Code plugin system — the outer `skills/` is the plugin root, the inner `skills/` is where the plugin system discovers skill definitions.

## The 10 Skills

- **implement** — 6-phase model (clarify → analyze → design → implement → test → review). Handles epics, parallel subagents, progressive bug escalation, session checkpoints.
- **code-review** — Systematic review against SOLID/Clean Code/DRY/KISS/YAGNI/security/performance. Iterative fix loop (max 3 rounds).
- **commit** — Discovers repos, analyzes diffs, suggests Conventional Commit messages. Detects split-worthy changes.
- **refine-ticket** — Transforms placeholder tickets into implementation-ready specs with data models, ASCII mockups, validation rules.
- **create-ticket** — Creates tickets from Product Owner perspective. Determines structure (single/multiple/epic).
- **create-adr** — Architecture Decision Records with alternatives and consequences.
- **project-status** — Reads planning file, shows milestone progress, recommends next ticket.
- **ideate** — Devil's advocate brainstorming. Challenges assumptions before creating tickets.
- **investigate** — Read-only diagnostics via Grafana MCP (Prometheus + Loki).
- **scaffold-feature** — "Scaffold by Example" — reads existing feature as template, replicates structure.

## Key Design Principles

- **Soft validation**: Skills check CLAUDE.md for required info and ask for it if missing — they never refuse to run.
- **Git is read-only**: Skills NEVER run `git commit`, `git push`, or other write-mode git commands.
- **No scope creep**: Implement exactly what was agreed, nothing more.
- **Build & test after every sub-task**: Every implementation change must compile and pass tests.
- **Phases are mandatory**: Skills follow ordered steps; no skipping phases.
- **Code first**: Read and understand existing patterns before writing new code.

## Writing and Editing Skills

When modifying a SKILL.md file:
- Keep phase/step numbering sequential and consistent
- Maintain the `# <SkillName>` / `## Phase N:` / `### Step N.M:` heading hierarchy
- Rules sections use numbered lists — these are hard constraints, not suggestions
- Each skill's plugin.json `description` field controls trigger phrase matching — keep it aligned with what the skill actually does
- Quality checklists (e.g., `quality-checklist.md`, `checklist.md`) are referenced by skills at review time — keep checklist categories stable

### Versioning

When a skill or plugin is changed, the `version` field in the corresponding `plugin.json` must be updated (semver):
- **patch** (0.0.x): Bug fixes, wording improvements, minor clarifications
- **minor** (0.x.0): New phases, new checklist categories, new behavior
- **major** (x.0.0): Breaking changes to skill flow or removed phases

Update the skill-level `plugin.json` (`skills/<name>/.claude-plugin/plugin.json`). If the root plugin is released, update `.claude-plugin/plugin.json` and `CHANGELOG.md` as well.

## Testing Changes

There is no automated test suite. To verify a skill change:
1. Install the plugin locally (reference the local path in `.claude/settings.json`)
2. Invoke the skill with a representative trigger phrase
3. Walk through the skill's phases to confirm correct behavior
4. Test edge cases: missing CLAUDE.md sections, empty repos, large diffs
