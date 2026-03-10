# CLAUDE.md Requirements — Detailed Reference

This document describes what each CLAUDE.md section should contain and which skills use it.
You only need to include sections relevant to the skills you use.

## Section: Project Overview

**Used by:** ideate, create-ticket, refine-ticket

Helps skills understand the project's purpose and make contextual decisions.

```markdown
## Project Overview
{Project name} is a {type of application} for {target audience}.
{1-2 sentences about core functionality.}

**Target audience:** {Who uses this}
**Non-goals:** {What this project explicitly does NOT do}
```

**Without this section:** `ideate` can't assess vision alignment. `create-ticket` can't judge priority.

---

## Section: Repository Structure

**Used by:** commit, implement, code-review, scaffold-feature

Tells skills how to discover and navigate the codebase.

```markdown
## Repository Structure
{Monorepo | Multi-repo}

- `{repo-or-dir}/` — {purpose}
- `{repo-or-dir}/` — {purpose}
```

**Examples:**

Monorepo:
```markdown
## Repository Structure
Monorepo with packages:
- `packages/api/` — Backend API (Node.js, Express)
- `packages/web/` — Frontend (React, Vite)
- `packages/shared/` — Shared types and utilities
```

Multi-repo:
```markdown
## Repository Structure
Multi-repo workspace. Each subdirectory is its own git repository:
- `myapp-api/` — Backend API (.NET 9, Minimal APIs)
- `myapp-web/` — Frontend (Blazor WASM)
- `myapp-infra/` — Infrastructure (Docker, CI/CD)
```

**Without this section:** `commit` scans for `.git` directories. `implement` may miss affected repos.

---

## Section: Build & Test Commands

**Used by:** implement, code-review, scaffold-feature

**This is the most critical section.** Skills need to know how to build and test your project.

```markdown
## Build & Test Commands

### {Project/Package 1}
```bash
{build command}
{test command}
```

### {Project/Package 2}
```bash
{build command}
{test command}
```
```

**Examples:**

```markdown
## Build & Test Commands

### API
```bash
dotnet build src/MyApp.Api.sln
dotnet test src/MyApp.Api.sln
```

### Web
```bash
cd packages/web && npm run build
cd packages/web && npm test
```
```

**Without this section:** `implement` will ask you for build/test commands before proceeding. `code-review` skips the build check step.

---

## Section: Tech Stack & Patterns

**Used by:** implement, code-review, refine-ticket, scaffold-feature

Describes the technologies and architectural patterns used. Skills reference this to follow conventions.

```markdown
## Tech Stack

| Layer | Technologies |
|-------|-------------|
| API | {language, framework, ORM, database} |
| Frontend | {framework, UI library, state management} |
| Infra | {containerization, CI/CD, hosting} |

## Architecture & Patterns

### {Layer} Patterns
- {Pattern 1}: {brief description}
- {Pattern 2}: {brief description}

### Naming Conventions
- {Convention 1}
- {Convention 2}

### Error Handling
- {Approach: Result types, exceptions, error codes, etc.}
```

**Without this section:** Skills use generic best practices. Project-specific conventions won't be enforced during review or implementation.

---

## Section: Planning & Tickets

**Used by:** create-ticket, refine-ticket, project-status, implement, ideate

Describes where tickets are stored and how they're structured.

```markdown
## Planning & Tickets

Planning lives in `{planning-directory}/`.

- **Planning file:** `{path to main planning file}` (milestones, ticket index)
- **Tickets:** `{path to tickets directory}/`
- **Archived tickets:** `{path to archive directory}/`
- **ADRs:** `{path to ADR directory}/`
- **Ticket ID prefix:** {PREFIX} (e.g., PROJ, MYAPP, TICKET)
```

**Without this section:** `create-ticket` and `project-status` will ask for the planning file location. `refine-ticket` can't find tickets to refine.

---

## Section: Infrastructure & Observability

**Used by:** investigate

Describes how to access environments and what monitoring is available.

```markdown
## Infrastructure

### Environments
| Environment | Access | Notes |
|-------------|--------|-------|
| Local | `docker compose up` | Direct access |
| Staging | SSH tunnel required | See below |

### Grafana / Monitoring
- **Local Grafana:** {URL}
- **Staging Grafana:** {URL or access method}

### Loki Labels
| Label | Values | Notes |
|-------|--------|-------|
| `{label}` | `{values}` | {purpose} |

### Key Prometheus Metrics
- `{metric_name}` — {what it measures}
```

**Without this section:** `investigate` will discover labels dynamically via `list_loki_label_names` / `list_prometheus_label_names`, but this is slower and may miss project-specific context.

---

## Section: Database Migrations

**Used by:** implement, scaffold-feature

Describes how database migrations work in your project.

```markdown
## Database Migrations
```bash
{migration command, e.g.:}
dotnet ef migrations add <Name> --project src/MyApp.Api
```

{Any special rules, e.g.: "Never edit migration files manually"}
```

**Without this section:** `implement` and `scaffold-feature` will remind you to handle migrations but can't tell you the exact command.

---

## Section: Git Conventions

**Used by:** commit, implement, code-review

Describes git workflow rules.

```markdown
## Git Conventions
- Commit format: Conventional Commits (`feat(scope): description`)
- {Any special rules, e.g.: "Always include ticket ID in scope"}
- {Branch naming convention}
```

**Without this section:** `commit` uses standard Conventional Commits. Git conventions are not project-specific.

---

## Quick Reference: Which Skills Need What

| CLAUDE.md Section | commit | code-review | implement | refine-ticket | create-ticket | project-status | ideate | investigate | scaffold-feature | create-adr |
|-------------------|:------:|:-----------:|:---------:|:-------------:|:-------------:|:--------------:|:------:|:-----------:|:----------------:|:----------:|
| Project Overview | | | | | R | | R | | | |
| Repo Structure | R | | R | | | | | | | |
| Build & Test | | **req** | **req** | | | | | | **req** | |
| Tech Stack | | R | R | R | | | | | R | |
| Architecture | | R | R | R | | | | | R | |
| Planning & Tickets | | | R | **req** | **req** | **req** | R | | | |
| ADR Directory | | | | | | | | | | **req** |
| Infrastructure | | | | | | | | **req** | | |
| DB Migrations | | | R | | | | | | R | |
| Git Conventions | R | | | | | | | | | |

**req** = Required (skill asks for this if missing)
**R** = Recommended (skill works without but at reduced quality)
