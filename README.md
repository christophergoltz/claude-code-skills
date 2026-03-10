# Claude Code Skills

A curated collection of reusable [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for software engineering workflows. These skills provide structured, repeatable processes for common development tasks — from implementation and code review to ticket management and infrastructure investigation.

## Installation

Add this plugin to your project's `.claude/settings.json`:

```json
{
  "plugins": [
    "https://github.com/christophergoltz/claude-code-skills"
  ]
}
```

Or install locally by cloning the repository and referencing the local path.

## Skills

| Skill | Description | Trigger phrases |
|-------|-------------|-----------------|
| **implement** | 6-phase implementation model: clarify → analyze → design → implement → test → review | `implement`, `fix bug`, `debug`, `work on`, `fix` |
| **code-review** | Systematic code review against SOLID, Clean Code, DRY, KISS, YAGNI, security, performance | `review`, `code review`, `check the code`, `review my changes` |
| **refine-ticket** | Transform placeholder tickets into implementation-ready specs with data models, mockups, validation rules | `refine`, `refinement`, `prepare ticket for implementation` |
| **commit** | Analyze git diffs across repos and suggest Conventional Commit messages | `commit`, `commit message`, `what changed`, `prepare commit` |
| **create-ticket** | Create planning tickets (simple or epic) from a Product Owner perspective | `create ticket`, `new ticket`, `add to backlog` |
| **create-adr** | Create Architecture Decision Records with alternatives and consequences | `create ADR`, `new ADR`, `document decision` |
| **project-status** | Show current milestone progress, open tickets, and next recommended action | `status`, `what's next`, `overview`, `progress` |
| **ideate** | Brainstorm and validate feature ideas through critical dialogue before they become tickets | `idea`, `brainstorm`, `what if we...`, `wouldn't it be cool if...` |
| **investigate** | Diagnose issues using Grafana (Prometheus + Loki) via MCP tools | `investigate`, `check logs`, `service down`, `why errors` |
| **scaffold-feature** | Generate a complete feature scaffold by reading an existing feature as reference | `scaffold`, `new feature`, `generate boilerplate` |

## CLAUDE.md Requirements

Skills read your project's `CLAUDE.md` to adapt to your codebase. Each skill needs specific information to function well. When a skill is invoked, it checks for the required information and will ask you to provide it if something is missing.

### Requirements by Skill

| Skill | Required | Recommended |
|-------|----------|-------------|
| **implement** | Build commands, test commands | Repo structure, ticket directory, tech stack, naming conventions |
| **code-review** | Build commands | Project patterns, naming conventions |
| **refine-ticket** | Ticket/planning directory | Tech stack, project patterns, repo structure |
| **commit** | — | Repo structure (monorepo vs multi-repo), commit conventions |
| **create-ticket** | Ticket directory with next-id, ticket ID prefix | Milestone structure |
| **create-adr** | ADR directory location | — |
| **project-status** | Planning file location | — |
| **ideate** | — | Planning directory, project vision/goals |
| **investigate** | Grafana MCP server configured | Service labels, environment access info |
| **scaffold-feature** | Build commands | Existing feature to use as reference |

**Required** = Skill cannot function without this. It will ask you to provide the info.
**Recommended** = Skill works without this but with reduced quality. It will note what's missing.

### Minimal CLAUDE.md Template

At minimum, your `CLAUDE.md` should contain the sections that your chosen skills need. Here's a template covering all skills:

```markdown
# Project Name

## Project Overview
{Brief description of the project, its purpose, and target audience.}

## Repository Structure
{Monorepo or multi-repo? List repositories and their purpose.}

## Tech Stack
{Languages, frameworks, key libraries.}

## Build & Test Commands
{How to build and test each part of the project.}

## Architecture & Patterns
{Key patterns used: vertical slices, CQRS, MVC, etc.
Naming conventions, error handling approach, etc.}

## Planning & Tickets
{Where tickets are stored, ID prefix, planning file location.
Where ADRs are stored.}

## Infrastructure
{Environments, how to access them, Grafana setup if applicable.}
```

See [docs/claude-md-requirements.md](docs/claude-md-requirements.md) for a detailed reference of what each section should contain and which skills use it. For advanced customization, see [docs/customization-guide.md](docs/customization-guide.md).

## How Skills Validate Prerequisites

Skills use a **soft validation** approach for good usability:

1. **Required info missing**: The skill reads CLAUDE.md at startup. If critical information is missing (e.g., no build commands for `implement`), it asks you via `AskUserQuestion` to provide the info before proceeding.

2. **Recommended info missing**: The skill notes what's missing and proceeds with sensible defaults or reduced functionality. For example, `code-review` works without a project patterns section but won't check project-specific conventions.

3. **No hard blockers**: Skills never refuse to run entirely. They guide you to provide what's needed, then continue. The exception is external dependencies (e.g., `investigate` needs a Grafana MCP server — if it's unreachable, the skill can't proceed).

This design prioritizes usability over strictness. You can start using skills immediately and fill in CLAUDE.md details over time as you discover what's useful.

## Customization

### Adding Project-Specific Patterns

Skills like `code-review` and `implement` reference generic checklists (`checklist.md`, `quality-checklist.md`). To add project-specific checks, document your patterns in CLAUDE.md:

```markdown
## Architecture & Patterns

### API Patterns
- Endpoints use `IEndpointDefinition` for auto-discovery
- Operations return `Result<T>` with `.ToMinimalApiResult()`
- Request validation via FluentValidation
- Mapping via Mapperly (not AutoMapper)

### Naming Conventions
- Async methods have `Async` suffix
- DTOs named `{Action}{Entity}Request` / `{Entity}Response`
```

Skills will read these and incorporate them into reviews and implementation.

### Overriding Skill Behavior

If a skill's default behavior doesn't fit your workflow, you can override specific aspects in CLAUDE.md:

```markdown
## Workflow Overrides
- Commits: Always include ticket ID in scope (e.g., `feat(PROJ-123): ...`)
- Code review: Skip performance section for prototype branches
- Implementation: No tests required for CSS-only changes
```

## Examples

The [`examples/`](examples/) directory contains complete, ready-to-copy references:

| Example | Description |
|---------|-------------|
| [claude-md-dotnet.md](examples/claude-md-dotnet.md) | CLAUDE.md for a .NET API + Blazor WASM project |
| [claude-md-node-react.md](examples/claude-md-node-react.md) | CLAUDE.md for a Node.js + React monorepo |
| [claude-md-python-fastapi.md](examples/claude-md-python-fastapi.md) | CLAUDE.md for a Python FastAPI + HTMX project |
| [planning-structure.md](examples/planning-structure.md) | Planning directory, PLANNING.md, tickets, and ADRs |
| [workflow-overrides.md](examples/workflow-overrides.md) | Workflow override patterns for all skills |

## License

Apache 2.0 — see [LICENSE](LICENSE).
