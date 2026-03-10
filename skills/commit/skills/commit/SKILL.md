---
name: commit
description: >
  Suggest conventional commit messages based on actual git diff analysis. Scans workspace
  repositories for changes, reads diffs, and proposes per-repo commit messages. Optionally
  uses ticket context for better scope descriptions. Detects when changes should be split
  into multiple commits. Use when the user says "commit", "commit message",
  "commit suggestions", "prepare commit", "what changed", "what should I commit",
  or asks about what changed or what to commit.
argument-hint: "[optional: ticket ID for scope context]"
allowed-tools: Bash(cd *, git status*, git diff*, git log*, echo *), Read, Glob, Grep
---

# Commit Skill

Analyze git changes across the workspace and suggest **Conventional Commit** messages.
You do NOT commit — you only suggest. Follow the project's git conventions from CLAUDE.md.

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Read CLAUDE.md for these. All are optional — the skill works without them but produces
better results when they're available.

| Info | Required | Used for |
|------|----------|----------|
| Repo structure | Recommended | Discovering git repositories in the workspace |
| Git/commit conventions | Recommended | Matching the project's commit style |
| Ticket ID prefix | Recommended | Including ticket IDs in commit scope |

## Step 1: Discover and Scan Repositories

Discover git repositories in the workspace. Check the project's CLAUDE.md for repository
structure hints. Common patterns:
- Monorepo: single `.git` at root
- Multi-repo: multiple subdirectories each with their own `.git`

Run `git status --short` in each repository (use parallel Bash calls when possible).
Skip any repository with no output.

## Step 2: Analyze Changes

For each repository with changes, run these two calls in parallel:

1. **Status + recent commits** — combine in one Bash call with a separator:
   ```bash
   git status --short && echo '---LOG---' && git log --oneline -5
   ```
   This single call delivers:
   - **Changed files** and their **staging state** (from the two-character status codes)
   - **Untracked files** (`??` prefix) — read their content with the `Read` tool
   - **Recent commit style** for consistency

   **Reading `git status --short` codes** (first two characters per line):
   | Code | Meaning |
   |------|---------|
   | `M·` | Staged (index modified) |
   | `·M` | Unstaged (worktree modified) |
   | `MM` | Both staged and unstaged changes |
   | `A·` | Newly staged file |
   | `??` | Untracked file |
   | `D·` | Staged deletion |
   | `·D` | Unstaged deletion |

   *(`·` represents a space character)*

   If any file shows mixed state (`MM`, or some files staged while others are not),
   flag this in the output (see Step 5).

2. **Diff content** — `git diff HEAD` to see all changes (staged + unstaged combined).
   If nothing is staged yet, `git diff HEAD` may fail — fall back to `git diff`.

## Step 3: Ticket Context (Optional)

If `$ARGUMENTS` contains a ticket ID:
- Read the ticket file from the planning directory described in CLAUDE.md
- Use the ticket's objective to write a more precise commit description
- The ticket ID goes into the **scope**: `feat(TICKET-123): add session notes`

If no ticket ID is provided, derive intent purely from the diff.

## Step 4: Compose Commit Messages

### Format

```
<type>(<scope>): <description>
```

**Types**: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`

### Scope Rules

Determine the scope in this priority order:

1. **Ticket ID provided** → use ticket ID as scope: `feat(TICKET-123): ...`
2. **No ticket** → derive from file paths — use the most prominent feature area or module name
3. **Cross-cutting changes** (e.g. renaming across many files) → use broad scope

### Description

- Concise, imperative mood, lowercase start
- Focus on the "what", not the "how"
- Full first line stays under ~72 characters
- English only

### Single-Line vs Multi-Line

**Single-line is the default.** Only use multi-line if there are genuinely diverse changes
that can't be summarized in one sentence:

```
<type>(<scope>): <summary>

- Detail 1
- Detail 2
```

### When to Suggest Splitting

If a single repository has changes touching **unrelated features** — for example a bug fix in
module A AND a new endpoint in module B — suggest splitting them into separate commits.

Only suggest splitting when the changes are genuinely independent — not when they're related
parts of one feature (e.g. endpoint + service + validator for the same feature belong together).

## Step 5: Present Output

Output ONLY the formatted suggestion below — no analysis, no explanation of staging state,
no commentary about why changes belong together. The commit message speaks for itself.

```
--- repo-name ---
feat(TICKET-123): add session notes CRUD endpoints

--- other-repo ---
chore(tickets): mark TICKET-123 as done
```

If only one repo has changes, still use the same format.

**Mixed staging state** — if a repo has both staged and unstaged changes, append a short warning
after the commit suggestion (this is the ONE exception where extra output is allowed):
```
⚠ repo-name has mixed staged/unstaged changes — commit staged only, or stage everything first?
```

**No changes found** — just say so, nothing more:
```
No changes found.
```

## Rules

- NEVER run `git commit`, `git push`, or any write-mode git commands
- NEVER guess changes — always read the actual diff first
- NEVER combine multiple repos into one commit message
- NEVER add commentary, analysis, or explanation around the output — the formatted suggestion is the entire response
- ALWAYS base the message on the actual diff, not session memory alone
- ALWAYS use English for commit messages
- ALWAYS use imperative mood ("add", "fix", "update" — not "added", "fixes")
