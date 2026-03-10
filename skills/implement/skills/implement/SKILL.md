---
name: implement
description: >
  Implement features, fix bugs, or debug issues following the 6-phase implementation model.
  Use when the user says "implement", "fix bug", "debug", "build", "develop", "work on",
  "fix", "resolve", or references a ticket ID.
argument-hint: "[optional: ticket ID or description, e.g. 'PROJ-149' or 'fix login redirect']"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, ToolSearch, Skill(code-review), AskUserQuestion, EnterPlanMode
---

# Implement Skill

You are implementing changes following the **6-phase implementation model**.
Work as a **senior developer** who knows the codebase, its patterns, and conventions.
Write production-ready code — no quick fixes, no placeholders, no TODOs as substitutes for real logic.

Read the project's CLAUDE.md to understand:
- Build and test commands
- Repository structure (monorepo vs multi-repo)
- Tech stack and patterns
- Naming conventions
- Ticket/planning system location

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, use `AskUserQuestion` to ask the user to provide it before proceeding.

| Info | Required | Used for |
|------|----------|----------|
| Build commands | **Yes** | Building after each sub-task |
| Test commands | **Yes** | Running tests after each sub-task |
| Repo structure | Recommended | Identifying affected repositories |
| Tech stack & patterns | Recommended | Following conventions, matching existing code |
| Ticket/planning directory | Recommended | Reading ticket specs (when triggered by ticket ID) |
| Naming conventions | Recommended | Consistent naming in new code |
| DB migration approach | Recommended | Reminding user of manual steps |

## Important Principles

1. **Follow the phases in order**: Do not skip phases. Each phase builds on the previous one.
2. **Read before writing**: ALWAYS understand existing code before modifying it.
3. **Stay on plan**: Implement exactly what was agreed in Phase 3. No scope creep.
4. **Minimal changes**: Only change what is necessary. No "while I'm here" refactorings.
5. **Match existing patterns**: Read reference code and follow the same conventions.
6. **Git is read-only**: NEVER run `git commit`, `git push`, or any write-mode git commands.
7. **Build and test are mandatory**: Every sub-task ends with a green build and passing tests.
   Skipping this is never acceptable, no matter how confident you are.

## IDE MCP Tools (Optional)

If the project has IDE MCP tools available (e.g. Rider, VS Code), load them via `ToolSearch`
in Phase 2. Prefer IDE-level analysis when available for faster feedback (e.g. instant file
problem checking vs. full build).

If IDE tools are not reachable, fall back to build/test commands from CLAUDE.md.

---

## Process

### Phase 1: Clarify Objective

**Goal**: Understand WHAT, HOW, and WHERE.

1. **If `$ARGUMENTS` contains a ticket ID**:
   - Read the ticket from the planning directory described in CLAUDE.md.
   - Extract: objective, acceptance criteria, affected areas, risks.

2. **If `$ARGUMENTS` contains a description** (e.g. `fix login redirect`):
   - Clarify with the user: What exactly is the problem/requirement? What is the expected behavior?
   - Identify affected areas.

3. **If requirements are unclear**:
   - Say **"STOP — can't start without clarity"** and list what needs clarification.
   - Do NOT proceed until requirements are clear.

4. **Refinement check** (for epics/large tickets):
   - If the ticket is an epic (has sub-tasks) and does NOT appear to be refined:
     - Warn: "This epic hasn't been refined yet. Unrefined tickets lead to worse
       implementation outcomes."
     - Use `AskUserQuestion` to ask: "Refine first?" with options:
       A) "Refine first (Recommended)"
       B) "Implement anyway"
     - If user chooses A: Stop and suggest running `/refine-ticket`
     - If user chooses B: Proceed (soft gate, not hard block)
   - Simple tickets (no sub-tasks, small scope): Skip this check

5. **Session scoping** (for epics — this is critical):

   Epics are too large for a single session. Trying to implement everything at once leads to
   lost context, skipped tests, and cascading failures. Always scope the work to a manageable chunk.

   **Steps:**
   - Read ALL sub-task files to understand sizes, dependencies, and what each involves.
   - Present a sub-task overview table to the user:
     ```
     Sub-Tasks for {ticket ID}:
     | # | Task | Size | Depends on | Status |
     |---|------|------|------------|--------|
     | 01 | ... | S | - | open |
     | 02 | ... | M | 01 | open |
     ```
   - Recommend which sub-tasks to implement this session based on:
     - **Size budget**: ~2-3 S/M tasks or ~1 L task per session
     - **Dependencies**: Only suggest tasks whose dependencies are met
     - **Independence**: Flag which tasks could run in parallel via subagents
   - Use `AskUserQuestion` to ask which sub-tasks to implement this session.
   - **Wait for the user's choice** before proceeding. Never assume "all of them".

   For simple tickets (not epics), skip this step entirely.

Answer these questions before moving on:
- What is the functional requirement?
- What are the acceptance criteria?
- What could go wrong?
- (Epics only) Which sub-tasks are we doing this session?

### Phase 2: Analyze Current State

**Goal**: Understand what EXISTS before implementing anything new.

1. **Load IDE MCP tools** (if available — try once, proceed without if unavailable).

2. **Identify affected areas**:
   - Which repositories/modules?
   - Which features within those?

3. **Read existing code**:
   - Entities, Services, Endpoints, Components — whatever is relevant
   - Understand data flows and current behavior
   - Note patterns to follow (naming, structure, error handling)

4. **Determine reuse vs. new implementation**:
   - What can be reused as-is?
   - What needs modification?
   - What must be built from scratch?

Present a brief summary of findings before moving to Phase 3.

### Phase 3: Solution Design

**Goal**: Clarify open design questions interactively, then create a detailed implementation plan.

**Step 1: Interactive Clarification**

Based on findings from Phase 2, identify any open design questions (e.g. approach trade-offs,
UI placement, naming, scope boundaries). Use `AskUserQuestion` to resolve them interactively:

- Ask all independent questions in a **single** `AskUserQuestion` call (up to 4 questions).
- If more than 4 questions are needed, batch them into multiple rounds.
- Provide concrete options with descriptions — always include a recommended option.
- If there are no open questions (trivial change with a clear path), skip directly to Step 2.

**Step 2: Enter Plan Mode**

After all questions are answered, switch to plan mode using `EnterPlanMode`.
In plan mode, create a detailed implementation plan with:

- Summary of the agreed approach (incorporating all answers from Step 1)
- **For epics: one plan section per selected sub-task**, in dependency order
- Ordered list of implementation steps per sub-task
- Affected files per step
- **Test plan**: Which tests to create or update (see below)
- Any risks or considerations

**Test Planning**: Proactively suggest tests as part of the plan. For each implementation step,
evaluate whether tests are warranted and include them in the plan:

| Change type | Test type | When to suggest |
|-------------|-----------|-----------------|
| New service/business logic | **Unit test** | Always — happy path + edge cases |
| New endpoint/route | **Unit test** for validation, **integration test** for full pipeline | Always for validation |
| Bug fix | **Unit test** that reproduces the bug | Always — prevents regression |
| New query/data access logic | **Unit test** | When query has filters, sorting, or joins |
| DTO mapping changes | **Unit test** | When mapping is non-trivial |
| Component logic | **Unit test** | When component has significant logic |
| Pure config / CSS / infra | None | Tests add no value here |

When suggesting tests, follow the existing test conventions from the project's test directory.

If no tests are warranted (e.g. pure config change), explicitly state "No tests needed" with
a brief justification — do not silently omit the section.

The plan format:
```
## Implementation Plan: [Feature/Fix Name]

**Approach**: Brief summary of the chosen approach
**Session Scope**: Sub-Tasks 01, 02, 03 (of 6)

### Sub-Task 01: [Name]
**Steps**:
1. [Step description]
   - Files: `path/to/file`
   - Details: What exactly to change/create

**Tests for Sub-Task 01**:
- [ ] Unit Test: `{TestClass}.{TestMethod}` — What it verifies

**Checkpoint**: Build + Test after Sub-Task 01

### Sub-Task 02: [Name]
...

**Parallelizable**: Sub-Tasks 03 + 04 have no shared dependencies
→ can be implemented as parallel subagents.

**Risks / Notes**:
- ...

**Affected Files**: ~X files total (incl. tests)
```

**Wait for user approval of the plan** before proceeding to Phase 4.

### Phase 4: Implementation

**Goal**: Execute the agreed plan precisely, one sub-task at a time.

**Core rules**:
- Implement exactly what was agreed in Phase 3
- No scope creep — no "while I'm here" changes
- Keep changes minimal and focused
- Follow existing patterns (see `quality-checklist.md` in this skill's directory)

#### Sub-Task Execution Protocol

For each sub-task in the approved plan, follow this cycle:

```
┌─────────────────────────────────┐
│  1. Implement sub-task          │
│  2. Build (MANDATORY)          │
│  3. Run tests (MANDATORY)      │
│  4. Fix failures               │
│  5. ✅ Checkpoint — inform user │
│  6. → Next sub-task            │
└─────────────────────────────────┘
```

**Per-file workflow**:
1. Read the file (understand context)
2. Make the change
3. Check for problems:
   - Prefer: IDE MCP tools for instant file-level feedback (if available)
   - Fallback: Build command after a batch of related changes

**After each sub-task is complete**:
1. Build the affected project(s) using commands from CLAUDE.md
2. Run tests for the affected project(s) using commands from CLAUDE.md
3. **Fix any build errors or test failures before moving to the next sub-task.**
   Do not proceed to the next sub-task with a broken build or failing tests.
4. Report to the user: "Sub-Task 01 complete. Build: green. Tests: X/X passed."

This checkpoint discipline prevents cascading failures. A bug introduced in sub-task 02
that is caught immediately costs minutes to fix. The same bug discovered 4 sub-tasks later
can cost hours or entire sessions.

#### Parallel Sub-Task Execution (Subagents)

When the plan identifies independent sub-tasks (no shared file changes, no dependency between
them), execute them in parallel using the `Task` tool:

**When to parallelize:**
- Sub-tasks touch different repos or different feature areas
- No sub-task depends on another's output
- Each sub-task is self-contained (own files, own tests)

**When NOT to parallelize:**
- Sub-tasks modify the same files
- One sub-task's code is needed by another
- Sub-tasks share a database migration

**How to dispatch:**

For each independent sub-task, spawn a Task subagent:

```
Task(subagent_type="general-purpose", prompt="""
You are implementing Sub-Task {N} of {Ticket ID}.

## Task
{Full sub-task spec from the ticket}

## Implementation Plan
{Relevant plan section from Phase 3}

## Context
- Working directory: {path}
- Affected area: {module/repo}
- Existing patterns to follow: {list key files to read as reference}

## Quality Checklist
{Paste or reference quality-checklist.md}

## Requirements
1. Read existing code before making changes
2. Follow existing patterns exactly
3. After implementation: build and test the affected project
4. Fix any build errors or test failures
5. Report: what was changed, build result, test result
""")
```

**After all subagents complete:**
1. Review each subagent's output — check for integration issues
2. Run a full build + test across ALL affected projects
3. Fix any cross-sub-task integration issues
4. Report combined results to the user

**Self-check during implementation** (from `quality-checklist.md`):
- SOLID principles followed
- Clean Code: descriptive names, short methods, no magic numbers
- DRY/KISS/YAGNI: no over-engineering, no unnecessary abstractions
- Error handling: appropriate patterns for the project
- Project patterns: follow conventions from CLAUDE.md and existing codebase
- Security: auth checks, input validation, no hardcoded secrets
- Performance: efficient queries, proper async usage

### Phase 5: Testing Handoff

**Goal**: Hand over to the user for manual testing with a session checkpoint for context safety.

#### Step 1: Write Session Checkpoint

Write a structured checkpoint file to `.claude/session-state.md` with the following content:

```markdown
# Session Checkpoint

## Ticket
- **ID**: {Ticket ID}
- **Title**: {Ticket title}

## Implemented Sub-Tasks
- [x] Sub-Task 01: {Name} — {brief description of what was done}
- [x] Sub-Task 02: {Name} — {brief description of what was done}

## Changed Files
- `path/to/file` — {what changed}

## Plan Summary
{2-3 sentences summarizing the agreed approach from Phase 3}

## Current Status
- **Build**: green/failing ({details})
- **Tests**: X/X passing / X failing ({details})

## Known Open Points
- {Any known limitations, edge cases not covered, or things to watch for}
- {Or "None" if none}
```

#### Step 2: Present Summary

1. **Summary**: What was changed this session, which files, why
2. **Build & test result**: Confirm green build and passing tests (with actual output)
3. **Manual steps** (if applicable):
   - Database migrations needed?
   - Infrastructure changes needed?
   - API spec regeneration needed?
   - App restart needed?
4. **Testing hints**: What the user should manually test and what to look for

#### Step 3: STOP — Wait for Feedback

End Phase 5 with:

> **Implementation complete. Please test and provide feedback — then we'll continue
> with the code review.**

Do NOT proceed to Phase 6 until the user confirms that testing is complete.

---

### Feedback Loop — Progressive Escalation

When the user reports bugs after Phase 5, escalate progressively:

```
User reports bug
├── Root cause obvious? (1-2 files, clear what's wrong)
│   ├── Yes → Level 1: Inline Fix
│   │   └── Fix successful? → Yes: Re-handoff (Phase 5 Step 2-3)
│   │                       → No: Level 2
│   └── No → Level 2: Subagent Fix
│       └── Fix successful? → Yes: Re-handoff (Phase 5 Step 2-3)
│                           → No: Level 3: Session Abort
```

#### Level 1 — Inline Fix (simple problems)

**When**: Obvious errors, typos, missing null-checks, small logic mistakes. Affects 1-2 files,
root cause is immediately clear.

**Action**: Fix directly in the main conversation, then:
1. Build + test
2. Update the session checkpoint (`.claude/session-state.md`)
3. Re-present the testing handoff (Phase 5 Step 2-3)

**Limit**: Maximum 1 inline fix attempt per bug. If the first attempt fails, escalate to Level 2.

#### Level 2 — Subagent Fix (medium problems)

**When**: After 1 failed inline fix attempt, OR when the problem requires investigation
(reading multiple files, tracing data flow, unclear root cause).

**Action**: Delegate the bug fix to a Task subagent with fresh context:

```
Task(subagent_type="general-purpose", prompt="""
You are fixing a bug in the codebase.

## Bug Description
{User's bug report}

## Context
{Read and paste relevant parts of .claude/session-state.md}

## Affected Files
{List the files most likely involved}

## Requirements
1. Read existing code before making changes
2. Follow existing patterns exactly
3. After fix: build and test the affected project
4. Fix any build errors or test failures
5. Report: root cause, what was changed, build result, test result
""")
```

**After subagent completes**:
1. Review the subagent's changes
2. Build + test in the main conversation
3. Update the session checkpoint
4. Re-present the testing handoff (Phase 5 Step 2-3)

#### Level 3 — Session Abort (complex/unknown problems)

**When**: After 1 failed subagent fix attempt, OR when the problem is fundamentally unclear
(race condition, environment-specific, deep architectural issue).

**Action**:
1. Document the bug in the session checkpoint (`.claude/session-state.md`):
   ```markdown
   ## Unresolved Bug
   - **Symptom**: {What was observed}
   - **Attempted Fixes**: {What was tried and why it failed}
   - **Suspected Cause**: {Best guess, or "unclear"}
   - **Recommendation**: Start a new session and focus specifically on this bug
   ```
2. Inform the user:
   > **This problem needs fresh context. Recommendation: Start a new session and
   > focus specifically on this bug. The session checkpoint contains all relevant info.**
3. End the implementation flow here. Do NOT proceed to Phase 6.

---

### Phase 6: Review & Finalization

**Goal**: Code review, ticket update, and commit preparation. Only after user confirms testing
is complete.

1. **Code Review**: Invoke the `code-review` skill to review the changes.
   - Run: `Skill(code-review)`
   - If the review finds **Critical** findings: fix them, then go back to the Feedback Loop
     (user needs to re-test the critical fixes).
   - If the review finds only **Improvement** or **Note** findings: present them to the
     user and ask whether to fix them or proceed as-is.

2. **Ticket update** (if implementation was triggered by a ticket ID):
   - Check CLAUDE.md for ticket management conventions (archive location, status field, etc.)
   - **If ALL sub-tasks of an epic are now done**: Mark ticket as done, move to archive
   - **If only some sub-tasks are done**: Update the ticket with progress, don't mark as done

3. **Commit**: Invoke the `/commit` skill for commit message suggestions.
   - Run: `Skill(commit)`
   - The commit skill will analyze the actual git diff and suggest messages.

4. **Next session preview** (for multi-session epics):
   - List remaining sub-tasks with sizes and dependencies
   - Suggest what to tackle next session

5. **Cleanup**: Delete `.claude/session-state.md` — the session is complete.

---

## Rules

- NEVER run `git commit`, `git push`, or any write-mode git commands.
- NEVER skip phases. Each phase builds on the previous one.
- NEVER implement without user confirmation on the approach (Phase 3).
- NEVER make changes beyond the agreed scope.
- NEVER skip build or test after a sub-task. This is the #1 cause of multi-session debugging pain.
- NEVER try to implement all sub-tasks of a large epic in one session without asking first.
- NEVER attempt more than 1 inline fix for the same bug before escalating to subagent.
- NEVER proceed to Phase 6 without user confirmation that testing is complete.
- ALWAYS read existing code before modifying it.
- ALWAYS verify the build compiles after each sub-task.
- ALWAYS run tests after each sub-task.
- ALWAYS write a session checkpoint before the testing handoff in Phase 5.
- ALWAYS delegate complex bug investigations to subagents to preserve main context.
- ALWAYS ask the user which sub-tasks to implement when working on an epic.
