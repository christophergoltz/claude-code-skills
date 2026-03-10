---
name: code-review
description: >
  Review code changes against SOLID, Clean Code, DRY, KISS, YAGNI, project patterns,
  security, and performance. Use when the user says "review", "code review",
  "check the code", "überprüfe den Code", "Code prüfen", "review my changes",
  "code-review", "review <number>" (PR review), or any variation involving
  reviewing, checking, or auditing code changes. ALWAYS use this skill when the
  user mentions "review" in the context of code, PRs, or changes — even with
  just a number like "review 187".
argument-hint: "[optional: specific focus area or file path]"
allowed-tools: Read, Glob, Grep, Edit, Write, EnterPlanMode, Bash(git diff*, git log*, git status*, dotnet build*, npm run build*, cargo build*, go build*, make build*)
---

# Code Review Skill

You are performing a systematic code review of the current changes.
Review from the perspective of a **senior developer** who knows the codebase, its
patterns, and conventions. Read the project's CLAUDE.md and codebase to understand
the established patterns before reviewing.

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, use `AskUserQuestion` to ask the user to provide it before proceeding.

| Info | Required | Used for |
|------|----------|----------|
| Build commands | **Yes** | Build check in Step 4 |
| Project patterns | Recommended | Project-specific review criteria |
| Naming conventions | Recommended | Verifying naming consistency |
| Tech stack | Recommended | Stack-specific checks (async patterns, ORM usage, etc.) |

## Important Principles

1. **Review only what changed**: Focus on the git diff (staged + unstaged). Do not review
   the entire codebase.
2. **Read surrounding context**: When a change touches a file, read enough of the file to
   understand the context (class structure, method signatures, imports).
3. **Be concrete**: Every finding must reference a specific file and line. Provide a
   concrete fix suggestion, not just "this could be improved".
4. **Prioritize correctly**: Not everything is critical. Use severity levels honestly.
5. **Respect existing patterns**: The project has established patterns (check CLAUDE.md).
   Flag deviations, but do not suggest rewriting code that follows existing conventions.
6. **Fix issues directly**: For Critical or Improvement findings, enter plan mode and fix
   the issues. Then re-review.

## Process

### Step 1: Gather Changes

Discover git repositories in the workspace (check CLAUDE.md for structure hints).
Run these commands to understand the scope:

```bash
git status
git diff --stat
git diff          # unstaged changes
git diff --cached # staged changes
```

If `$ARGUMENTS` specifies a file or focus area, narrow the review scope accordingly.

### Step 2: Read the Diff

Read the full diff output carefully. Identify:
- Which files were changed
- What kind of changes (new feature, bugfix, refactoring, etc.)
- The approximate scope (how many files, how many lines)

### Step 3: Read Context

For each changed file, read the full file (or relevant sections) to understand:
- The class/component structure
- How the changed code fits into the larger context
- What patterns the file follows

### Step 4: Build Check

Run the build command for the project. Check CLAUDE.md for the correct build command.
Common patterns:
- .NET: `dotnet build <SolutionFile>`
- Node.js: `npm run build`
- Rust: `cargo build`
- Go: `go build ./...`

Look for:
- Compiler/build errors
- New warnings introduced by the changes

Compare warnings before/after if possible. Focus on **new** warnings.

### Step 5: Systematic Review

Review the changes against the checklist (see `checklist.md` in this skill's directory).
Go through each category in order:

1. **Compiler & Warnings** — from Step 4 results
2. **SOLID Principles** — structural design quality
3. **Clean Code** — readability and naming
4. **DRY / KISS / YAGNI** — simplicity and duplication
5. **Error Handling** — robustness
6. **Project Patterns** — project-specific conventions from CLAUDE.md and codebase
7. **Security** — authorization, input validation, data exposure
8. **Performance** — queries, async, allocations

For each finding, note:
- **Category** (from above)
- **Severity**: Critical / Improvement / Note
- **File and line**
- **Description** of the problem
- **Suggestion** for how to fix it

**Output Rule**: Only report categories that have findings. Do NOT list categories
that are clean — no "Clean", no "No issues". Silent means clean. If the user
asks about a specific category, you may comment on it.

### Step 6: Act on Results

Handle findings based on their severity:

**No Findings** → Just one sentence: "No findings — looks good." Done.
No listing of individual categories.

**Notes only** → List notes briefly (file, line, description). Ask the user if
these should be addressed. Only fix with approval.

**Critical and/or Improvement** →
1. List all findings (file, line, category, description)
2. Enter plan mode (`EnterPlanMode`) and create a fix plan
3. After approval, implement the fixes
4. Continue to Step 7 (re-review)

### Step 7: Review Loop (Until Clean)

After fixing:
1. Re-read the diff (`git diff`)
2. Check build
3. Repeat systematic review (Step 5) — only on the changed parts
4. If Critical/Improvement findings remain → fix again (back to Step 6)
5. If only Notes or no findings → report to the user and finish

Maximum 3 iterations. After that, list remaining findings and let the user decide.

## Severity Guidelines

**Critical** — Must be fixed before commit/merge:
- Compiler errors or new warnings
- Security vulnerabilities (missing auth, SQL injection, XSS)
- Data loss risk (missing null checks on external input, wrong delete behavior)
- Broken existing functionality
- Deadlock risk (blocking on async operations)

**Improvement** — Should be fixed, but not a blocker:
- SOLID violations that hurt maintainability
- Code duplication (DRY violation)
- Missing input validation
- Deviation from established project patterns (check CLAUDE.md)
- Performance anti-patterns (N+1 queries, unnecessary allocations)

**Note** — Optional improvement, nice-to-have:
- Naming could be more expressive
- Method is getting long
- Minor simplification possible
- Inconsistent style within the change

## Rules

- Fixes are implemented exclusively via plan mode (`EnterPlanMode`) — never directly
  without prior planning and approval.
- NEVER fix code that was not part of the diff (unless it is directly broken by the changes).
- NEVER run `git commit`, `git push`, or any write-mode git commands.
- ALWAYS read the actual diff before making any claims about the code.
- If the diff is empty (no changes), inform the user and stop.
