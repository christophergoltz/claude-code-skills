---
name: refine-ticket
description: >
  Refine a planning ticket to implementation-ready detail level. Transforms placeholder
  sub-tasks into detailed specifications with data models, ASCII mockups, validation rules,
  and edge cases. Use this skill whenever the user wants to add detail to a ticket, prepare
  a ticket for implementation, or flesh out sub-tasks — even if they don't use the word "refine"
  explicitly. Trigger phrases include: "refine", "refinement", "refine-ticket", "prepare ticket
  for implementation", "flesh out sub-tasks", "add detail to ticket", "detail the ticket".
argument-hint: "[ticket ID, e.g. 'PROJ-175']"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git status*, git diff*), AskUserQuestion, ToolSearch
---

# Refine Ticket Skill

You are refining a planning ticket to **implementation-ready detail level**. Your job is to
transform placeholder sub-tasks (10-20 lines) into detailed specifications (50-130+ lines per
sub-task) with data model specs, ASCII mockups, validation rules, edge cases, and integration points.

Every ambiguity must be resolved. Every requirement must be explicit. The goal: a developer (or
Claude) can implement the ticket without asking a single clarifying question.

Read the project's CLAUDE.md to understand:
- Planning/ticket directory structure
- Tech stack and patterns
- Repository structure
- Naming conventions

**Communicate in the user's language.** Detect from their messages and respond accordingly.
Keep YAML frontmatter keys and Markdown section headers always in English.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, use `AskUserQuestion` to ask the user to provide it before proceeding.

| Info | Required | Used for |
|------|----------|----------|
| Ticket/planning directory | **Yes** | Finding and writing ticket files |
| Tech stack & patterns | Recommended | Informed questions about data models, APIs, UI |
| Repo structure | Recommended | Identifying which areas the ticket affects |
| Architecture patterns | Recommended | Proactive analysis of missing requirements |

## Important Principles

1. **NEVER write code** — only ticket/planning files.
2. **ALWAYS read the codebase** before asking questions — ask targeted, informed questions.
3. **ALWAYS confirm with the user** before writing any sub-task files.
4. **Max 2-4 questions per round** — never dump all questions at once.
5. **Proactive suggestions** — don't just ask, propose solutions based on codebase knowledge.
6. **Git is read-only**: NEVER run `git commit`, `git push`, or any write-mode git commands.

## IDE MCP Tools (Optional)

Try loading IDE MCP tools via `ToolSearch` in Step 3 for deeper codebase exploration
(symbol info, dependency graphs). If not reachable, fall back to Glob/Grep/Read — these
are sufficient for refinement.

---

## Process

### Step 1: Load Ticket

1. Extract ticket ID from `$ARGUMENTS`.
2. Try to read the ticket from the planning directory described in CLAUDE.md:
   - Simple ticket: single file
   - Epic: overview file + sub-task files
3. If `$ARGUMENTS` contains a description instead of an ID:
   - Suggest the **"Create + Refine" flow**: First create the ticket structure (user should
     run `/create-ticket` first), then refine it.
   - Do NOT create the ticket yourself — that is the `create-ticket` skill's job.
4. If the ticket is not found: **"STOP — Ticket not found: {ID}"**

### Step 2: Assess Refinement Scope

Evaluate the ticket's current state and determine the refinement approach:

| Condition | Action |
|-----------|--------|
| Already refined (marked in frontmatter) | Inform user. Offer to re-refine specific sub-tasks if needed. |
| Simple ticket (XS/S, no sub-tasks) | **Quick-Check mode**: Validate against quality checklist, ask 1-2 clarifying rounds max. |
| Epic with placeholder sub-tasks | **Full refinement**: Estimate effort to user ("This epic has {N} sub-tasks to refine. This will take {N} Q&A rounds.") |
| Epic with partially refined sub-tasks | **Partial refinement**: Only refine sub-tasks that lack detail. Skip already-detailed ones. |

Present the assessment to the user and confirm before proceeding.

### Step 3: Explore Codebase

**Critical**: Read existing code BEFORE asking any questions. This enables targeted, informed
questions instead of generic ones.

1. **Read planning context** from the planning system described in CLAUDE.md:
   - Which milestone does this ticket belong to?
   - Are there related tickets that affect scope or ordering?
   - Are there dependencies on tickets that are not yet refined or implemented?
     If so, flag this to the user.

2. **Load IDE MCP tools** (try once, proceed without if unavailable).

3. **Based on ticket scope, read relevant code**:

   | Ticket type | What to read |
   |-------------|--------------|
   | Data model changes | Models, database configs, existing feature slices |
   | API endpoints | Endpoints, DTOs, validators, services/queries |
   | Frontend component | Components, layout, theme, CSS, JS interop |
   | Auth changes | Auth controllers, services, OAuth config |
   | Infrastructure | Docker, proxy configs, CI/CD workflows |

4. **Present findings as summary**:
   ```
   Relevant patterns/entities/components:
   - [Entity X] already uses [Pattern Y] — apply same pattern
   - [Component Z] uses [Library W] — stay consistent
   - Existing validation: [approach used in Feature A]
   ```

### Step 4: Proactive Analysis

Based on codebase knowledge and ticket content, identify:

1. **Inconsistencies** in the ticket:
   - Wrong dependencies between sub-tasks
   - Missing sub-tasks for obviously needed steps
   - Scope gaps (e.g., frontend ticket without API endpoint)

2. **Missing requirements** (based on codebase patterns):
   - Audit/soft-delete considerations?
   - Sort order / ordering logic needed?
   - API spec update needed?
   - Input validation rules?
   - Query filters (user isolation, soft-delete)?

3. **UX gaps** (for frontend tickets):
   - Empty state defined?
   - Loading state defined?
   - Error state defined?
   - Dark/light mode considered?
   - Responsive design considered?
   - Keyboard navigation?
   - Accessibility?

4. **Integration points** with existing features:
   - Which existing components/services can be reused?
   - Are there side effects on other features?

Present everything as a structured list to the user.

### Step 5: Iterative Q&A Rounds

Resolve all open questions through focused, iterative rounds:

**Rules:**
- **Max 2-4 questions per round** via `AskUserQuestion`
- **Always provide a recommended option** (list first or mark as recommended)
- After each round: incorporate answers, identify follow-up questions, start next round
- Continue until ALL ambiguities are resolved

**Question categories** (use as needed based on ticket scope):

1. **Data Model**: Columns, types, constraints, indexes, FK behavior, soft-delete, defaults
2. **UX / UI Design**: Layout, interaction, states (empty/loading/error/success), responsive, animations
3. **API Design**: Endpoints, HTTP methods, DTOs, authorization, pagination, sorting, filtering, errors
4. **Validation**: Constraints per field, error messages, client-side vs. server-side
5. **Edge Cases**: Concurrent access, deletion of referenced entities, limits, offline behavior
6. **Integration**: Existing systems, reuse vs. new, backward compatibility
7. **Migration**: Existing data handling, reversibility, feature flags

### Step 6: Design Decisions Summary

After all Q&A rounds are complete, compile all decisions:

1. **Data model design**: Table schemas with columns, types, constraints
2. **UX specification**: ASCII mockups, icon mappings, responsive breakpoints
3. **API design**: Endpoint overview, DTO structures
4. **Migration strategy**: Existing data handling
5. **Integration points**: Reuse decisions, affected existing features

Present the complete summary to the user and get confirmation before writing files.

### Step 7: Write Refined Sub-Tasks

After user confirmation, write the detailed sub-task files.

**Template for each sub-task:**

```markdown
# {NN}: {Task Title}

## Goal

{1-3 sentences describing what this sub-task achieves and why it matters.}

## Dependencies

- Requires: {list of sub-tasks that must be completed before this one}
- Blocks: {list of sub-tasks that depend on this one}

## Requirements

### {Section 1 — e.g., Data Model}

- [ ] {Specific, implementable requirement}
- [ ] {Another requirement}

### {Section 2 — e.g., API Endpoint}

- [ ] {Requirement}
- [ ] {Requirement}

### {Section 3 — e.g., Validation}

- [ ] {Requirement}
- [ ] {Requirement}

{3-6 sections depending on sub-task scope}

## Estimated Size

{XS | S | M | L | XL}

## Notes

- {Implementation hint, gotcha, or pattern to follow}
- {Reference to existing code: "See `features/X/` for reference pattern"}
- {Edge case to watch out for}
```

**Minimum detail requirements scale with ticket size** (see `quality-checklist.md` for thresholds).
As a guideline for content per sub-task type:

| Sub-task type | Key sections to include |
|---------------|------------------------|
| Data model | Schema (columns, types, constraints), indexes, query filters, soft-delete, seed data |
| API endpoints | Paths, HTTP methods, auth, request/response DTOs, error codes, pagination |
| Frontend component | ASCII-mockup, component parameters, events, states (empty/loading/error), responsive |
| Migration / data changes | Before/after state, data transformation, rollback strategy |

**Workflow:**
1. Draft all sub-tasks in the conversation
2. Present them to the user for review
3. Incorporate feedback
4. Write the files only after confirmation

### Step 8: Update Frontmatter

After writing all sub-task files:

1. Set `refined: true` in the YAML frontmatter of the overview file
2. Update `updated: {today's date}` in the frontmatter
3. Remove any placeholder notes from the overview

### Step 9: Quality Checklist

Validate the refined ticket against the quality checklist (see `quality-checklist.md` in this
skill's directory).

**All items must either pass or be explicitly marked as N/A with justification.**

Run through each category:
- Data Model
- UX / UI
- API
- Validation
- Integration
- Edge Cases
- Sub-Task Quality

If any items fail, fix them before completing.

### Step 10: Completion

1. **Summary**: "{N} sub-tasks refined to implementation-ready level"
2. **Suggest next step**: "`/implement {ticket ID}` as next step"
3. **Commit message suggestions**: Follow the [Commit Message Suggestions](#commit-message-suggestions)
   section below.

---

## Commit Message Suggestions

After all files are written, suggest commit messages based on the **actual git status** —
not just what this skill changed.

1. Run `git status` in the relevant repository/repositories.
2. Group changes by repository.
3. For each repository with changes, suggest a **separate** commit message.
4. Use **Conventional Commits** format (English):
   ```
   <type>(<scope>): <description>
   ```
   Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`
5. Commit messages are **single-line** by default.
6. **Do NOT commit** — only suggest. Git is read-only.

---

## Rules

- NEVER write code — only ticket/planning files.
- NEVER write sub-task files without user confirmation.
- NEVER ask generic questions — always read the codebase first.
- NEVER dump all questions in one round — max 2-4 per `AskUserQuestion` call.
- NEVER skip the codebase exploration step.
- NEVER mark as refined without completing the quality checklist.
- ALWAYS provide recommended options in questions.
- ALWAYS include ASCII mockups for frontend sub-tasks.
- ALWAYS specify all states (empty/loading/error) for UI components.
- ALWAYS validate against quality-checklist.md before completing.
- ALWAYS suggest `/implement` as the next step.
- Git is read-only — NEVER run write-mode git commands.
