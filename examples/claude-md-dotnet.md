# Example CLAUDE.md — .NET API + Blazor WASM

This example shows a complete CLAUDE.md for a .NET project with a backend API and Blazor frontend.
Copy the sections you need and adapt them to your project.

---

```markdown
# MyProject

## Project Overview

MyProject is a web application for managing inventory and orders.

**Target audience:** Small business owners
**Non-goals:** Marketplace features, B2B integrations

## Repository Structure

Multi-repo workspace. Each subdirectory is its own git repository:
- `myproject-api/` — Backend API (.NET 9, Minimal APIs, PostgreSQL)
- `myproject-web/` — Frontend (Blazor WASM, MudBlazor)
- `myproject-infra/` — Infrastructure (Docker Compose, GitHub Actions)
- `myproject-planning/` — Tickets, ADRs, milestones

## Build & Test Commands

### API
```bash
dotnet build myproject-api/src/MyProject.Api.sln
dotnet test myproject-api/src/MyProject.Api.sln
dotnet run --project myproject-api/src/MyProject.Api
```

### Web
```bash
dotnet build myproject-web/src/MyProject.Web.sln
dotnet test myproject-web/tests/MyProject.Web.Tests
dotnet run --project myproject-web/src/MyProject.Web
```

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| API | .NET 9, Minimal APIs, EF Core 9, PostgreSQL, FluentValidation, Mapperly |
| Web | Blazor WASM, MudBlazor, Refit |
| Infra | Docker Compose, Nginx, GitHub Actions |
| Observability | Serilog, OpenTelemetry, Prometheus, Loki, Grafana |

## Architecture & Patterns

### API (Feature-Sliced)
```
src/MyProject.Api/
├── Features/
│   └── {Feature}/
│       ├── {Feature}Endpoints.cs
│       ├── {Feature}Service.cs
│       ├── {Feature}Queries.cs
│       ├── DTOs/
│       └── Validation/
├── Models/
├── Data/
└── Configuration/
```

### Key Patterns
- `IEndpointDefinition` interface for auto-discovered endpoint registration
- `Result<T>` for operation results with `.ToMinimalApiResult()` extension
- Mapperly for compile-time DTO mapping
- Global query filters for soft deletes and user isolation

### Naming Conventions
- Endpoints: `{Feature}Endpoints` implementing `IEndpointDefinition`
- Services: `{Feature}Service` for write operations
- Queries: `{Feature}Queries` for read operations
- DTOs: `Create{Entity}Request`, `Update{Entity}Request`, `{Entity}Response`
- Validators: `Create{Entity}RequestValidator`

### Error Handling
- Services return `Result<T>` (Ardalis.Result)
- Validation via FluentValidation
- No exceptions for control flow

## Database Migrations
```bash
dotnet ef migrations add <MigrationName> --project myproject-api/src/MyProject.Api
```

Never edit migration files manually.

## Planning & Tickets

Planning lives in `myproject-planning/`.
- **Planning file:** `myproject-planning/PLANNING.md`
- **Tickets:** `myproject-planning/tickets/`
- **Archived tickets:** `myproject-planning/tickets/archive/`
- **ADRs:** `myproject-planning/decisions/`
- **Ticket ID prefix:** MYPROJ

## Infrastructure

### Environments
| Environment | Access |
|-------------|--------|
| Local | `docker compose up` |
| Staging | SSH tunnel required |

### Grafana / Monitoring
- **Local Grafana:** http://localhost:3000

## Git Conventions
- Commit format: Conventional Commits
- Always include ticket ID in scope: `feat(MYPROJ-42): add export`
- Git is read-only for Claude — never commit or push

## Critical Rules
- Never use `:latest` for Docker image tags
- All entities inherit `AuditableEntity` for soft delete
- No business logic in endpoint definitions
- Enums over magic strings for known value sets
```
