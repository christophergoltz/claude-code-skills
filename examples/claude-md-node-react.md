# Example CLAUDE.md — Node.js API + React Frontend

This example shows a complete CLAUDE.md for a TypeScript monorepo with Express backend and React frontend.
Copy the sections you need and adapt them to your project.

---

```markdown
# TaskFlow

## Project Overview

TaskFlow is a project management tool for small development teams.

**Target audience:** Dev teams of 2-15 people
**Non-goals:** Enterprise features (SSO, audit logs, compliance)

## Repository Structure

Monorepo with pnpm workspaces:
- `packages/api/` — Backend API (Node.js, Express, Prisma)
- `packages/web/` — Frontend (React 19, Vite, TanStack Query)
- `packages/shared/` — Shared types and utilities
- `packages/db/` — Database schema and migrations (Prisma)
- `planning/` — Tickets, ADRs, milestones

## Build & Test Commands

### API
```bash
pnpm --filter api build
pnpm --filter api test
pnpm --filter api dev    # Start dev server
```

### Web
```bash
pnpm --filter web build
pnpm --filter web test
pnpm --filter web dev    # Start dev server
```

### All packages
```bash
pnpm build       # Build all
pnpm test        # Test all
pnpm lint        # Lint all
pnpm typecheck   # Type-check all
```

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| API | Node.js 22, Express, Prisma, PostgreSQL, Zod |
| Web | React 19, Vite, TanStack Query, TanStack Router, Tailwind CSS |
| Shared | TypeScript, shared Zod schemas |
| Infra | Docker Compose, GitHub Actions, Fly.io |
| Testing | Vitest, Testing Library, MSW |

## Architecture & Patterns

### API (Route-based)
```
packages/api/src/
├── routes/
│   └── {resource}/
│       ├── {resource}.routes.ts    # Express router
│       ├── {resource}.service.ts   # Business logic
│       ├── {resource}.schema.ts    # Zod validation schemas
│       └── {resource}.test.ts      # Tests
├── middleware/    # Auth, error handling, logging
├── lib/           # Shared utilities (prisma client, logger)
└── index.ts       # App entry point
```

### Web (Feature-based)
```
packages/web/src/
├── features/
│   └── {feature}/
│       ├── components/         # Feature-specific components
│       ├── hooks/              # Feature-specific hooks
│       ├── {feature}.page.tsx  # Page component
│       └── {feature}.api.ts    # API calls (TanStack Query)
├── components/    # Shared UI components
├── hooks/         # Shared hooks
├── lib/           # Utilities, API client
└── main.tsx
```

### Key Patterns
- Zod schemas shared between API validation and frontend forms
- TanStack Query for server state with optimistic updates
- Error responses follow RFC 7807 (Problem Details)
- All API routes require authentication except `/auth/*`

### Naming Conventions
- Files: kebab-case (`user-profile.tsx`)
- Components: PascalCase (`UserProfile`)
- Hooks: camelCase with `use` prefix (`useUserProfile`)
- API routes: plural nouns (`/api/v1/projects`, `/api/v1/tasks`)
- Tests: co-located with source (`*.test.ts`, `*.test.tsx`)

### Error Handling
- API: Express error middleware catches all errors, returns RFC 7807
- Zod validation errors auto-mapped to 400 with field-level details
- Frontend: Error boundaries per feature, toast notifications for mutations

## Database Migrations
```bash
pnpm --filter db migrate:dev     # Create + apply migration
pnpm --filter db migrate:deploy  # Apply pending migrations (prod)
pnpm --filter db generate        # Regenerate Prisma client
```

## Planning & Tickets

Planning lives in `planning/`.
- **Planning file:** `planning/PLANNING.md`
- **Tickets:** `planning/tickets/`
- **Archived tickets:** `planning/tickets/archive/`
- **ADRs:** `planning/decisions/`
- **Ticket ID prefix:** TF

## Git Conventions
- Commit format: Conventional Commits
- Include ticket ID: `feat(TF-15): add drag-and-drop for tasks`
- Branch naming: `feat/TF-15-drag-and-drop`, `fix/TF-22-login-redirect`

## Workflow Overrides
- Implementation: Always run `pnpm lint` and `pnpm typecheck` after changes
- Code review: Check that Zod schemas are shared (not duplicated between API and web)
- Commits: Suggest separate commits for API and web changes
```
