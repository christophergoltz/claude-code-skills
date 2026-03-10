---
name: project-status
description: >
  Show the current project status from the planning file. Displays completed milestones,
  current milestone progress, and open tickets. Use when the user asks "status",
  "where are we", "what's next", "overview", "project status", "next ticket",
  "progress", or any similar status/overview question about the project.
allowed-tools: Read, Glob
---

# Project Status Skill

Read the project's planning file (as described in CLAUDE.md) and output the project status
in a structured format. Do not deviate from this format.

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, inform the user what's needed and stop.

| Info | Required | Used for |
|------|----------|----------|
| Planning file location | **Yes** | Reading milestones and ticket status |

## Steps

1. Read CLAUDE.md to find where the planning file is located.
2. Read the planning file fully.
3. Parse all milestone sections.
4. For each milestone, count tickets by status from the milestone table.
5. Identify the **current milestone**: the first milestone that is NOT fully done — i.e.,
   has at least one ticket that is not `done` or `cancelled`.
6. Output the status in the format below.

## Output Format

Use this template. Replace placeholders with actual data.

```
# {Project Name} — Project Status

**Completed:** v0.4, v0.5, v0.6
**Current milestone:** v0.7 — {Milestone Name}

## v0.7 — {Milestone Name} (X/Y Tickets done)

| # | Ticket | Title | Priority | Size | Refined | Dependencies |
|---|--------|-------|----------|------|---------|--------------|
| 1 | TICKET-187 | Task description | high | L | Yes | — |
| 2 | TICKET-188 | Other task | high | Epic | No | TICKET-187 |

**Next step:** TICKET-187 — Task description (high, L, refined)
```

## Format Rules

1. **Completed**: List all milestones that are fully done, comma-separated.
2. **Current milestone**: The first non-done milestone. Show version + name.
3. **Ticket Table**:
   - Only show tickets that are NOT `done` and NOT `cancelled`.
   - Sort by priority: `critical` > `high` > `medium` > `low`.
   - Within same priority, keep the order from the planning file.
   - **#**: Sequential row number.
   - **Ticket**: Just the ID, no link.
   - **Title**: Ticket title from the milestone table.
   - **Priority**: Priority value.
   - **Size**: Size value.
   - **Refined**: `Yes` if refined, `No` if not.
   - **Dependencies**: Comma-separated ticket IDs from the `dependencies` field, or `—` if none.
     To find dependencies, check the ticket's YAML frontmatter if the milestone table doesn't show them.
4. **Next step**: The highest-priority ticket that is refined and has no unfinished dependencies.
   This is the recommended next ticket to work on. If no ticket is refined and unblocked,
   say "No ticket is refined and unblocked — refine a ticket first."
5. If a ticket is `in-progress`, mark it in the **Next step** line.
6. If ALL tickets in the current milestone are done, show the milestone as complete and move
   to the next one.

## Important

- Do NOT add commentary, suggestions, or explanations beyond the format.
- Do NOT suggest which ticket to work on beyond the "Next step" line.
- Do NOT read individual ticket files unless needed to resolve dependencies.
- The output should be quick and scannable — that's the whole point of this skill.
