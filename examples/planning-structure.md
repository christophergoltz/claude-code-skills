# Example Planning Structure

This example shows how to set up the planning directory used by `project-status`, `create-ticket`, `refine-ticket`, and `ideate`.

---

## Directory Structure

```
planning/
├── PLANNING.md              # Main planning file (milestones, ticket index)
├── CONTEXT.md               # Optional: project vision, current state
├── tickets/
│   ├── PROJ-001.md          # Active tickets
│   ├── PROJ-002.md
│   ├── PROJ-003.md
│   └── archive/
│       ├── PROJ-000.md      # Completed tickets
│       └── ...
└── decisions/
    ├── ADR-001-auth-strategy.md
    └── ADR-002-database-choice.md
```

---

## PLANNING.md Example

```markdown
# ProjectName — Planning

## Milestones

### v0.1 — Foundation ✅
Core infrastructure and authentication.

| ID | Title | Priority | Status |
|----|-------|----------|--------|
| [PROJ-001](tickets/archive/PROJ-001.md) | Project setup and CI/CD | high | done |
| [PROJ-002](tickets/archive/PROJ-002.md) | User authentication | critical | done |
| [PROJ-003](tickets/archive/PROJ-003.md) | Base UI layout | medium | done |

### v0.2 — Core Features (current)
First usable feature set.

| ID | Title | Priority | Status |
|----|-------|----------|--------|
| [PROJ-004](tickets/PROJ-004.md) | CRUD for projects | critical | done |
| [PROJ-005](tickets/PROJ-005.md) | Task management | critical | in progress |
| [PROJ-006](tickets/PROJ-006.md) | Dashboard overview | high | open |
| [PROJ-007](tickets/PROJ-007.md) | Search functionality | medium | open |

### v0.3 — Collaboration
Multi-user features.

| ID | Title | Priority | Status |
|----|-------|----------|--------|
| [PROJ-008](tickets/PROJ-008.md) | Team invitations | critical | open |
| [PROJ-009](tickets/PROJ-009.md) | Activity feed | high | open |
| [PROJ-010](tickets/PROJ-010.md) | Comments on tasks | medium | open |

## Backlog

Unscheduled tickets, roughly prioritized.

| ID | Title | Priority | Status |
|----|-------|----------|--------|
| [PROJ-011](tickets/PROJ-011.md) | Email notifications | medium | open |
| [PROJ-012](tickets/PROJ-012.md) | Dark mode | low | open |
| [PROJ-013](tickets/PROJ-013.md) | CSV export | low | open |
```

---

## Ticket Example (before refinement)

```markdown
# PROJ-006: Dashboard Overview

**Priority:** high
**Size:** M
**Status:** open

## Goal

Show users a summary dashboard when they log in. Should display recent activity,
their active tasks, and key metrics.

## Sub-Tasks

1. Design dashboard layout
2. Implement recent activity widget
3. Implement active tasks widget
4. Implement metrics widget

## Acceptance Criteria

- Dashboard loads within 2 seconds
- Shows last 10 activity items
- Shows user's open tasks sorted by due date
- Shows project-level metrics (total tasks, completed, overdue)

## Dependencies

- PROJ-004 (projects CRUD) must be done
- PROJ-005 (task management) must be done
```

---

## Ticket Example (after refinement with `/refine`)

A refined ticket has detailed sub-tasks with data models, API specs, UI mockups, validation rules, and edge cases. The `refine-ticket` skill transforms placeholder sub-tasks into implementation-ready specifications.

```markdown
# PROJ-006: Dashboard Overview

**Priority:** high
**Size:** M
**Status:** refined

## Goal

Show users a summary dashboard when they log in. Should display recent activity,
their active tasks, and key metrics.

## Sub-Tasks

### Sub-Task 1: Dashboard API Endpoint

**Goal:** Create GET endpoint returning all dashboard data in one request.

**Dependencies:** PROJ-004, PROJ-005

**Endpoint:**
- `GET /api/v1/dashboard`
- Auth: Required (returns data scoped to authenticated user)

**Response DTO:**
```json
{
  "recentActivity": [
    {
      "id": "uuid",
      "type": "task_created | task_completed | comment_added",
      "description": "Created task 'Fix login bug'",
      "projectName": "MyProject",
      "timestamp": "2025-01-15T10:30:00Z"
    }
  ],
  "activeTasks": [
    {
      "id": "uuid",
      "title": "Fix login bug",
      "projectName": "MyProject",
      "dueDate": "2025-01-20",
      "priority": "high"
    }
  ],
  "metrics": {
    "totalTasks": 42,
    "completedTasks": 28,
    "overdueTasks": 3
  }
}
```

**Query logic:**
- `recentActivity`: Last 10 activities for user's projects, ordered by timestamp DESC
- `activeTasks`: User's open tasks, ordered by due date ASC (nulls last)
- `metrics`: Aggregated counts across user's projects

**Edge cases:**
- New user with no projects → return empty arrays and zero metrics
- Tasks without due date → sort after tasks with due dates
- Activity from deleted projects → exclude (soft-delete filter)

**Estimated Size:** S

---

### Sub-Task 2: Dashboard Page UI

**Goal:** Build the dashboard page with three widget cards.

**Dependencies:** Sub-Task 1

**Layout:**
```
┌─────────────────────────────────────────────────┐
│  Welcome back, {username}              [v0.2.0] │
├────────────────────┬────────────────────────────┤
│                    │                            │
│  Recent Activity   │     My Tasks               │
│  ─────────────     │     ────────               │
│  • Created "Fix…"  │  ☐ Fix login bug  Jan 20  │
│    2 hours ago     │  ☐ Add export     Jan 22  │
│  • Completed "…"   │  ☐ Update docs    —       │
│    5 hours ago     │                            │
│  • Comment on "…"  │                            │
│    yesterday       │                            │
│                    │                            │
├────────────────────┴────────────────────────────┤
│  Project Metrics                                │
│  ───────────────                                │
│  Total: 42    Completed: 28    Overdue: 3       │
│  ████████████████████░░░░░░░░  67%              │
└─────────────────────────────────────────────────┘
```

**States:**
- **Loading:** Skeleton cards with pulsing animation
- **Empty:** "No activity yet. Create your first project to get started."
- **Error:** "Could not load dashboard. [Retry]"

**Responsive:**
- Desktop (>1024px): 2-column layout as shown
- Tablet/Mobile (<1024px): Single column, stacked

**Edge cases:**
- Long task titles → truncate with ellipsis after 50 chars
- Overdue tasks → show due date in red
- No due date → show "—" instead of date

**Estimated Size:** M

## Acceptance Criteria

- Dashboard loads within 2 seconds
- Shows last 10 activity items
- Shows user's open tasks sorted by due date
- Shows project-level metrics (total tasks, completed, overdue)
- All three states handled (loading, empty, error)
- Responsive layout works on mobile
```

---

## ADR Example

```markdown
# ADR-001: Authentication Strategy

**Status:** Accepted
**Date:** 2025-01-10

## Context

We need user authentication for the application. Users should be able to register,
log in, and maintain sessions across page reloads.

## Decision

Use JWT tokens with short-lived access tokens (15 min) and long-lived refresh tokens
(7 days). Store refresh tokens in HTTP-only cookies. Access tokens are sent as
Bearer tokens in the Authorization header.

## Alternatives Considered

### Session-based authentication
- **Pro:** Simpler, well-understood, easy to revoke
- **Con:** Requires server-side session store, harder to scale horizontally

### OAuth2 with external provider only (Google, GitHub)
- **Pro:** No password management, trusted providers
- **Con:** Not all users have Google/GitHub, vendor dependency

## Consequences

- Need a token refresh endpoint
- Need secure cookie configuration for refresh tokens
- Need token rotation on refresh (invalidate old refresh token)
- Access tokens are stateless — can't revoke individual tokens (acceptable given 15 min TTL)
```
