---
name: create-adr
description: >
  Create a new Architecture Decision Record (ADR). Use when the user says "new ADR",
  "create ADR", "architecture decision", "document decision", "ADR anlegen".
argument-hint: "[optional: brief description of the decision]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git status*, git diff*)
---

# Create ADR Skill

You are creating an Architecture Decision Record (ADR) for the project.

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, use `AskUserQuestion` to ask the user to provide it before proceeding.

| Info | Required | Used for |
|------|----------|----------|
| ADR directory location | **Yes** | Finding existing ADRs and writing new ones |

## Important Principles

1. **Draft before writing**: Always present the full ADR in the conversation and get user
   confirmation before writing any files.
2. **Alternatives Considered is mandatory**: Never skip this section. Include at least
   "Do nothing" as an alternative if no others are obvious.
3. **Status default**: Use `proposed` unless the user explicitly says the decision is already
   accepted.
4. **Today's date**: Always use today's date for the `date` field.

## Process

### Step 1: Gather Information

Read the user's request. You need:
- **Decision topic**: What architectural decision is being made? (required — from user input or $ARGUMENTS)
- **Context**: Why is this decision needed? What problem are we solving?
- **Alternatives**: What options were considered?
- **Decision**: Which option was chosen and why?

If the user provides a clear description, proceed directly. If the description is vague,
ask focused clarifying questions about the decision context and alternatives.

### Step 2: Determine Next Number and Location

Read the project's CLAUDE.md to find where ADRs are stored (typically a `decisions/` directory
in a planning folder). Use Glob to find existing ADRs and determine the next number.

Find the highest ADR number and increment by 1. Format as 3-digit padded number (e.g., `ADR-004`).

### Step 3: Read Reference Material

Read the ADR template (if one exists) and 1-2 existing ADRs to match the established
writing style and structure.

### Step 4: Draft the ADR

Present the complete ADR in the conversation. Follow this structure:

```markdown
---
id: ADR-{NNN}
title: "{Decision title}"
status: proposed
date: {YYYY-MM-DD}
ticket: {ticket ID if applicable, otherwise empty}
supersedes: {ADR-XXX if applicable, otherwise empty}
superseded-by:
---

# ADR-{NNN}: {Decision Title}

## Context

{Why is this decision needed? What is the current situation or problem?}

## Decision

{What was decided? Describe the chosen approach in detail.}

## Alternatives Considered

### Alternative A: {Name}

**Approach:** {Brief description}

**Pros:**
- ...

**Cons:**
- ...

### Alternative B: Do Nothing

**Approach:** Keep the current state.

**Pros:**
- No effort required
- No risk of breaking changes

**Cons:**
- {Why doing nothing is not acceptable}

## Consequences

### Positive

- ...

### Negative

- ...

## Sources

- {Links to relevant documentation, discussions, or references}
```

### Step 5: User Confirmation

Present the draft and ask for confirmation.
Wait for confirmation. Incorporate any requested changes before writing.

### Step 6: Write the Files

After user confirmation:

1. **Generate slug**: Create a kebab-case slug from the title (e.g., `entity-model-restructuring`)
2. **Write ADR file**: Write to the decisions directory found in Step 2
3. **Update index** (if applicable): If the project has a context/index file listing ADRs, update it

### Step 7: Handle Supersedes (if applicable)

If the new ADR supersedes an existing one:

1. Read the old ADR file
2. Update the old ADR's frontmatter:
   - Set `superseded-by: ADR-{NNN}` (the new ADR's ID)
   - Set `status: superseded`
3. Update any index/context file accordingly

### Step 8: Confirm Completion and Suggest Commits

Tell the user the ADR was created successfully and show the file path.

Then suggest commit messages based on the actual `git status`. Discover git repositories
in the workspace and suggest per-repo commit messages using Conventional Commits format.
Do NOT commit — only suggest. Follow the project's git conventions from CLAUDE.md.

## Rules

- NEVER write an ADR without presenting the draft first.
- NEVER skip the "Alternatives Considered" section.
- NEVER set status to anything other than `proposed` unless the user explicitly says "accepted".
- ALWAYS read existing ADRs to determine the correct next number.
- ALWAYS suggest commit messages after successful completion.
- Use today's date for the `date` field.
