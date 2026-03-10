# Example CLAUDE.md — Python FastAPI + HTMX

This example shows a complete CLAUDE.md for a Python project with FastAPI backend and HTMX frontend.
Copy the sections you need and adapt them to your project.

---

```markdown
# DataHub

## Project Overview

DataHub is an internal data catalog and quality monitoring tool.

**Target audience:** Data engineers and analysts within the organization
**Non-goals:** Public-facing UI, real-time streaming, data transformation/ETL

## Repository Structure

Single repository:
- `src/datahub/` — Application code
- `tests/` — Test suite
- `migrations/` — Alembic database migrations
- `planning/` — Tickets, ADRs, milestones
- `infra/` — Docker, Terraform, CI/CD

## Build & Test Commands

```bash
# Install dependencies
uv sync

# Run tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=datahub --cov-report=term-missing

# Run linting
uv run ruff check src/ tests/
uv run mypy src/

# Start dev server
uv run uvicorn datahub.main:app --reload --port 8000
```

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| API | Python 3.13, FastAPI, SQLAlchemy 2.0, PostgreSQL, Pydantic v2 |
| Frontend | Jinja2 templates, HTMX, Alpine.js, Tailwind CSS |
| Infra | Docker, Terraform (AWS), GitHub Actions |
| Testing | pytest, httpx (async test client), factory_boy |

## Architecture & Patterns

### Application Structure
```
src/datahub/
├── features/
│   └── {feature}/
│       ├── router.py        # FastAPI router
│       ├── service.py       # Business logic
│       ├── models.py        # SQLAlchemy models
│       ├── schemas.py       # Pydantic schemas
│       ├── repository.py    # Database queries
│       └── templates/       # Jinja2 templates (HTMX partials)
├── core/
│   ├── config.py            # Settings (pydantic-settings)
│   ├── database.py          # Engine, session factory
│   ├── dependencies.py      # FastAPI dependencies
│   └── security.py          # Auth middleware
├── templates/               # Base templates, layouts
└── main.py                  # App factory
```

### Key Patterns
- Repository pattern for database access (no raw queries in services)
- Pydantic v2 schemas for all request/response validation
- FastAPI dependency injection for services and repositories
- HTMX partials for interactive UI without client-side JS framework
- Background tasks via FastAPI BackgroundTasks (simple) or Celery (complex)

### Naming Conventions
- Files: snake_case (`data_source.py`)
- Classes: PascalCase (`DataSource`, `DataSourceCreate`)
- Functions: snake_case (`get_data_source`)
- Schemas: `{Entity}Create`, `{Entity}Update`, `{Entity}Response`
- Routes: plural nouns (`/api/v1/data-sources`)

### Error Handling
- Custom exception classes inheriting from `AppError`
- Global exception handler returns consistent JSON error responses
- Pydantic validation errors auto-mapped to 422 with field details
- Services raise domain exceptions, routers don't catch them (handler does)

## Database Migrations
```bash
# Create migration
uv run alembic revision --autogenerate -m "description"

# Apply migrations
uv run alembic upgrade head

# Rollback one migration
uv run alembic downgrade -1
```

Never edit migration files manually after they've been applied.

## Planning & Tickets

Planning lives in `planning/`.
- **Planning file:** `planning/PLANNING.md`
- **Tickets:** `planning/tickets/`
- **Archived tickets:** `planning/tickets/archive/`
- **ADRs:** `planning/decisions/`
- **Ticket ID prefix:** DH

## Infrastructure

### Environments
| Environment | URL | Access |
|-------------|-----|--------|
| Local | http://localhost:8000 | Direct |
| Staging | https://staging.datahub.internal | VPN required |

### Grafana / Monitoring
- **Staging Grafana:** https://grafana.internal:3000

### Loki Labels
| Label | Values | Notes |
|-------|--------|-------|
| `app` | `datahub` | Application name |
| `env` | `local`, `staging`, `prod` | Environment |
| `level` | `DEBUG`, `INFO`, `WARNING`, `ERROR` | Log level |

## Git Conventions
- Commit format: Conventional Commits
- Include ticket ID: `feat(DH-8): add data lineage graph`

## Workflow Overrides
- Implementation: Always run `uv run ruff check` and `uv run mypy src/` after changes
- Code review: Verify that all new endpoints have OpenAPI descriptions
- Commits: Single repo, so one commit per logical change
```
