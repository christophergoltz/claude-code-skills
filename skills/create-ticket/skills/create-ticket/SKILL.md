---
name: create-ticket
description: >
  Create a new planning ticket (simple or epic). Use when the user wants to
  create a ticket, add a new feature request, log a bug, file an enhancement,
  or says things like "create a ticket", "new ticket", "add to backlog",
  or "ticket for [feature]".
argument-hint: "[optional: brief description of the ticket]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git status*, git diff*)
---

# Create Ticket Skill

You are creating a planning ticket from a **Product Owner perspective**.

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, use `AskUserQuestion` to ask the user to provide it before proceeding.

| Info | Required | Used for |
|------|----------|----------|
| Ticket/planning directory | **Yes** | Finding planning file, writing ticket files |
| Ticket ID prefix | **Yes** | Generating ticket IDs (e.g., PROJ-001) |
| Milestone structure | Recommended | Placing tickets in the right milestone |

## Important Principles

1. **Product Owner perspective**: Write tickets that describe WHAT and WHY, never HOW.
   Do NOT include implementation details, code examples, file paths, database columns,
   or technical architecture. Those come later during implementation planning.
2. **Present before writing**: Always draft the ticket in the conversation first and ask
   the user to confirm before writing any files.
3. **Minimal questioning**: Do not ask unnecessary questions. If you can infer the type
   (feature/bugfix/etc.) and priority from context, do so. Only ask if the user's
   description is genuinely unclear or ambiguous.

## Process

### Step 1: Gather Information

Read the user's request. You need:
- **Description**: What the ticket is about (required — from user input or $ARGUMENTS)
- **Type**: feature | enhancement | bugfix | chore | docs (infer from context, default: feature)
- **Priority**: critical | high | medium | low (default: medium)
- **Size**: XS | S | M | L | XL (optional — estimate from context, do not actively ask)

If the user provides a clear description, proceed directly. If the description is vague
or ambiguous, ask 1-2 focused clarifying questions about the **functional requirements**.
Do NOT ask about technical details.

**Scope Assessment — automatically evaluate the right ticket structure:**

After understanding the requirements, assess which structure fits:

1. **Single simple ticket** — the scope is clearly bounded and can be implemented in one work session.
   Proceed directly to drafting.
2. **Multiple separate tickets** — the user describes unrelated, independent topics in one request
   (e.g., "I need an export feature and also the login is broken"). Recommend creating separate tickets,
   explain why (different concerns), and after confirmation process each one sequentially.
3. **Epic with sub-tasks** — a coherent feature that is too large for a single ticket. Indicators:
   - Multiple functional areas affected (e.g., backend + frontend + auth)
   - Clear natural boundaries between steps
   - Scope is realistically not implementable atomically

If the assessment results in option 2 or 3, present the recommendation to the user with a brief
justification before proceeding. The user makes the final decision.

### Step 2: Read Current State

Read the project's planning file (as described in CLAUDE.md) to get the next ticket ID.
Look for a `next-id` field in YAML frontmatter or determine the next ID from existing tickets.

### Step 3: Draft the Ticket

Compose the full ticket content internally, but do NOT show it to the user yet.
Instead, present a **compact summary** for verification (see Step 4).

#### Simple Ticket Format

```markdown
---
id: {PREFIX}-{ID}
title: "{Title}"
status: backlog
type: {type}
priority: {priority}
size: {size}
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
dependencies: []
decision:
milestone:
---

# {PREFIX}-{ID}: {Title}

## Context

{Why does this ticket exist? What is the current situation or user problem?}

## Goal

{What is the desired outcome from the user's perspective? Be concrete and specific.}

## User Story

{Optional — only include if it naturally fits the ticket. Remove section entirely if not applicable.
Format: As a [role], I want [capability], so that [benefit].}

## Requirements

- [ ] {Functional, user-facing requirement 1}
- [ ] {Requirement 2}
- [ ] {Requirement 3}

## Acceptance Criteria

- [ ] {Given [precondition], when [action], then [expected result]}
- [ ] {Criterion 2}

## Out of Scope

{What is explicitly NOT part of this ticket? Helps prevent scope creep.}

## Open Questions

{Questions that need answering before or during implementation. Remove section entirely if none.}
```

#### Epic Overview Format

For epics, create an overview file (e.g. `00-overview.md`) in a ticket directory:

```markdown
---
id: {PREFIX}-{ID}
title: "{Epic Title}"
status: backlog
type: {type}
priority: {priority}
size: {size}
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
decision:
milestone:
---

# {PREFIX}-{ID}: {Epic Title}

**Project:** {Brief description of what this epic achieves}
**Status:** Backlog

---

## Context

{Why does this epic exist? What is the current situation?}

## Goal

{What is the desired outcome from the user's perspective?}

## Scope

{What is included and what is explicitly excluded?}

## Task Overview

| # | Task | File | Dependencies | Status |
|---|------|------|--------------|--------|
| 01 | {Task name} | `01-{slug}.md` | - | [ ] |
| 02 | {Task name} | `02-{slug}.md` | 01 | [ ] |

> Sub-tasks are placeholders. Before implementation: run `/refine-ticket {PREFIX}-{ID}`.

---

## Acceptance Criteria

- [ ] {High-level acceptance criterion for the entire epic}
- [ ] {Criterion 2}

## Open Questions

{Questions that need answering before planning begins. Remove section entirely if none.}
```

### Step 4: User Confirmation

The user doesn't want to read the full ticket — they want to quickly verify that you understood
the right problem and scope. Present a **compact summary**, not the full ticket content.

**Summary format:**

```
{PREFIX}-{ID}: {Title}
{type} | {priority} | {size} | Milestone: {milestone}

Core idea: {Problem} -> {Solution}

{N} Requirements | {N} Acceptance Criteria | Out of Scope: {1-line summary}
```

The "Core idea" line is the most important part. It must follow the pattern **Problem -> Solution**:
explain WHY this ticket exists (the pain point) and THEN what we do about it.

After the summary, ask for confirmation. Wait for confirmation. Incorporate any requested
changes before writing.

If the user asks to see the full ticket before writing, show it — but don't do so by default.

### Step 5: Write the Files

After user confirmation:

1. **Simple ticket**: Write file to the tickets directory
2. **Epic**: Create a ticket directory and write `00-overview.md` inside it.

Then update the planning index file:
1. Add a new row to the appropriate milestone table
2. Increment `next-id` in the YAML frontmatter

**File locations**: Read from CLAUDE.md — typical patterns:
- Active tickets: `{planning-dir}/tickets/{PREFIX}-{ID}.md` (or `tickets/{PREFIX}-{ID}/` for epics)
- Archived tickets: `{planning-dir}/tickets/archive/{PREFIX}-{ID}.md`

### Step 6: Confirm Completion

Tell the user the ticket was created successfully and show the file path.

### Step 7: Suggest Refinement (for Epics)

If the ticket is an epic (has sub-tasks), suggest running the refine-ticket skill.

## Commit Message Suggestions

After all files are written (Step 5/6), suggest commit messages based on the **actual
git status** — not just what this skill changed. The user may have made additional changes.

1. Discover git repositories in the workspace.
2. Group changes by repository.
3. For each repository with changes, suggest a **separate** commit message.
4. Use **Conventional Commits** format (English):
   ```
   <type>(<scope>): <description>
   ```
   Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`
5. Commit messages are **single-line** by default.
6. **Do NOT commit** — only suggest. Follow the project's git conventions from CLAUDE.md.

## Rules

- NEVER write implementation details, code snippets, file paths, or architecture decisions into the ticket.
- NEVER change the status to anything other than `backlog` for new tickets.
- NEVER create a ticket without reading the current `next-id` first.
- ALWAYS present the draft to the user before writing files.
- ALWAYS suggest commit messages after successful completion.
- Use today's date for `created` and `updated` fields.
- When creating multiple tickets in one session, process them sequentially — read `next-id` fresh before each ticket.
