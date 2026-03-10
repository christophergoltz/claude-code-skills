# Customization Guide

This guide explains how to adapt claude-code-skills to your project. Skills are designed to be project-agnostic — they read your `CLAUDE.md` at runtime and adjust their behavior accordingly.

## Table of Contents

- [How Skills Discover Your Project](#how-skills-discover-your-project)
- [CLAUDE.md as Configuration](#claudemd-as-configuration)
- [Customizing Individual Skills](#customizing-individual-skills)
  - [implement](#implement)
  - [code-review](#code-review)
  - [refine-ticket](#refine-ticket)
  - [commit](#commit)
  - [create-ticket](#create-ticket)
  - [project-status](#project-status)
  - [investigate](#investigate)
  - [scaffold-feature](#scaffold-feature)
  - [ideate](#ideate)
  - [create-adr](#create-adr)
- [Overriding Skill Behavior](#overriding-skill-behavior)
- [Adding Project-Specific Checklists](#adding-project-specific-checklists)
- [Multi-Repo Workspaces](#multi-repo-workspaces)

---

## How Skills Discover Your Project

Skills follow this order to understand your project:

1. **CLAUDE.md** — Primary source. Skills look for specific sections (see [claude-md-requirements.md](claude-md-requirements.md)).
2. **File system exploration** — If CLAUDE.md doesn't provide enough info, skills use `Glob` and `Read` to discover project structure.
3. **Interactive questions** — For required info that can't be discovered, skills ask you via `AskUserQuestion`.

The more complete your CLAUDE.md, the faster and more accurate skills behave.

---

## CLAUDE.md as Configuration

Your `CLAUDE.md` is the primary configuration mechanism. Skills don't use separate config files — they read the same CLAUDE.md that Claude Code already uses.

### Minimum Viable CLAUDE.md

```markdown
# My Project

## Build & Test Commands
```bash
npm run build
npm test
```
```

This is enough to use `implement` and `code-review`. Other skills will ask for what they need.

### Recommended Sections

See [claude-md-requirements.md](claude-md-requirements.md) for the full reference. The most impactful sections are:

1. **Build & Test Commands** — Unlocks `implement`, `code-review`, `scaffold-feature`
2. **Planning & Tickets** — Unlocks `create-ticket`, `refine-ticket`, `project-status`
3. **Architecture & Patterns** — Improves quality of `implement`, `code-review`, `refine-ticket`

---

## Customizing Individual Skills

### implement

The implementation skill follows a 6-phase model. You can influence its behavior through CLAUDE.md:

**Naming conventions** — Document these so generated code follows your patterns:
```markdown
## Architecture & Patterns
### Naming Conventions
- Services: `{Entity}Service`
- DTOs: `{Action}{Entity}Request` / `{Entity}Response`
- Tests: `{ClassName}Tests` with methods `{Method}_Should{Expected}_When{Condition}`
```

**Test requirements** — Specify what should be tested:
```markdown
## Testing
- Unit tests required for all service methods
- Integration tests for API endpoints
- No tests needed for pure DTO/mapping changes
```

**Implementation rules** — Add constraints the skill should follow:
```markdown
## Critical Rules
- Never use `DateTime.Now` — always inject `IDateTimeProvider`
- All database queries must use async/await
- No business logic in controllers/endpoints
```

### code-review

The review skill uses a built-in checklist (`checklist.md`) covering SOLID, Clean Code, DRY, KISS, YAGNI, security, and performance. You can extend it:

**Project-specific patterns to enforce:**
```markdown
## Architecture & Patterns
### Must-Follow Patterns
- All API responses wrapped in `Result<T>`
- Validation via FluentValidation, not manual checks
- Mapping via Mapperly, not AutoMapper
- No `string` for known value sets — use enums
```

**Things to skip:**
```markdown
## Workflow Overrides
- Code review: Skip performance section for prototype branches
- Code review: Don't flag missing XML docs in internal services
```

### refine-ticket

Refine transforms vague tickets into implementation-ready specs. It produces better results when it knows your tech stack:

```markdown
## Tech Stack
| Layer | Technologies |
|-------|-------------|
| API   | Node.js, Express, Prisma, PostgreSQL |
| Web   | React 19, TanStack Query, Tailwind CSS |
```

This lets refine generate accurate data models, API contracts, and component structures in your stack.

### commit

Commit scans for git changes and suggests Conventional Commit messages. Customize via:

```markdown
## Git Conventions
- Always include ticket ID in scope: `feat(PROJ-123): add export`
- Use `chore` for dependency updates
- Multi-repo: suggest separate commits per repository
```

**Multi-repo setup** — If your workspace has multiple git repos, document it:
```markdown
## Repository Structure
Multi-repo workspace:
- `my-api/` — Backend
- `my-web/` — Frontend
```

The commit skill will scan each repo independently and suggest separate commit messages.

### create-ticket

The ticket creation skill needs to know your planning system:

```markdown
## Planning & Tickets
- **Planning file:** `planning/PLANNING.md`
- **Tickets:** `planning/tickets/`
- **Ticket ID prefix:** MYAPP
- **Next ticket ID:** Check `planning/PLANNING.md` for the highest existing ID
```

**Ticket templates** — The skill creates tickets with a standard structure (Goal, Sub-Tasks, Acceptance Criteria, Dependencies). If you need additional sections, add them to your workflow:

```markdown
## Workflow Overrides
- Tickets: Always include an "Impact Assessment" section
- Tickets: Add "Rollback Plan" for infrastructure changes
```

### project-status

Needs to know where your planning file is:

```markdown
## Planning & Tickets
- **Planning file:** `planning/PLANNING.md`
```

The skill reads your milestone tables and presents current progress. The planning file should use markdown tables with columns for ticket ID, title, priority, and status.

### investigate

Requires a Grafana MCP server. Configure it in `.mcp.json`, then document your setup:

```markdown
## Infrastructure

### Grafana / Monitoring
- **Local Grafana:** http://localhost:3000

### Loki Labels
| Label | Values | Notes |
|-------|--------|-------|
| `service_name` | `my-api`, `my-web` | Main application services |
| `env` | `local`, `staging`, `prod` | Environment |

### Key Prometheus Metrics
- `http_request_duration_seconds` — Request latency histogram
- `http_requests_total` — Request counter by status code
```

This lets investigate query the right labels and metrics immediately instead of discovering them dynamically.

### scaffold-feature

Scaffold reads an existing feature in your codebase as a template, then generates a new feature following the same structure. You control what gets generated by documenting your feature structure:

```markdown
## Architecture & Patterns

### Feature Structure (API)
Each feature lives in `src/Features/{FeatureName}/`:
- `{Feature}Endpoints.cs` — Route definitions
- `{Feature}Service.cs` — Business logic
- `{Feature}Queries.cs` — Read operations
- `DTOs/` — Request/response models
- `Validation/` — FluentValidation validators
```

When invoked, scaffold will ask which existing feature to use as reference and what the new feature should be called.

### ideate

Works best when it knows your project vision:

```markdown
## Project Overview
MyApp is a project management tool for small teams.
**Target audience:** Freelancers and small agencies (1-20 people)
**Non-goals:** Enterprise features, SSO, compliance tools
```

This lets ideate assess whether a feature idea aligns with the project's direction.

### create-adr

Needs to know where ADRs are stored:

```markdown
## Planning & Tickets
- **ADRs:** `planning/decisions/`
```

The skill creates ADRs with a standard template: Context, Decision, Alternatives Considered, Consequences.

---

## Overriding Skill Behavior

You can override specific aspects of any skill by adding a `Workflow Overrides` section to CLAUDE.md:

```markdown
## Workflow Overrides
- Commits: Always include ticket ID in scope
- Code review: Skip performance section for prototype branches
- Implementation: No tests required for CSS-only changes
- Implementation: Always run `npm run lint` after changes
- Tickets: Use German for ticket descriptions
```

Skills read this section and adjust their behavior. Overrides are free-form — describe the change in natural language and the skill will interpret it.

---

## Adding Project-Specific Checklists

The `implement` and `code-review` skills use built-in quality checklists. To extend them with project-specific checks, you have two options:

### Option 1: Document in CLAUDE.md (recommended)

Add patterns and rules directly to your CLAUDE.md. Skills read these and incorporate them:

```markdown
## Critical Rules
- Never expose internal IDs in API responses — use public slugs
- All file uploads must go through the `StorageService`
- Database queries must not use `ToList()` before filtering
```

### Option 2: Create a local checklist file

Create a checklist file in your project and reference it:

```markdown
## Workflow Overrides
- Code review: Also check against `docs/review-checklist.md`
- Implementation: Verify against `docs/quality-gates.md` before completion
```

---

## Multi-Repo Workspaces

If your project spans multiple repositories in a single workspace directory:

```markdown
## Repository Structure
Multi-repo workspace. Each subdirectory is its own git repository:
- `myapp-api/` — Backend API
- `myapp-web/` — Frontend
- `myapp-infra/` — Infrastructure

Git commands must be run inside each repository subdirectory.
```

Skills that interact with git (`commit`, `implement`) will handle each repo independently:
- `commit` scans each repo for changes and suggests separate commit messages
- `implement` runs build/test commands in the correct repo directory
- `code-review` checks changes across all affected repos

---

## Tips

- **Start small.** Begin with Build & Test Commands and add sections as needed.
- **Be specific.** `"Use Result<T> for all service returns"` is better than `"Use result types"`.
- **Keep it current.** Update CLAUDE.md when patterns change. Outdated info is worse than no info.
- **Use examples.** When documenting naming conventions, include concrete examples.
