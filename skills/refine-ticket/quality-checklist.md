# Refine Ticket — Quality Checklist

Use this checklist to validate that a refined ticket meets implementation-ready quality.
All items must pass or be explicitly marked as **N/A** with justification.

Adapt this checklist to the project's tech stack. Check CLAUDE.md for project-specific
patterns and conventions that may add requirements.

## Scope-based Thresholds

Not every ticket needs the same level of detail. Apply thresholds based on ticket size:

| Ticket Size | Min. lines/sub-task | Min. edge cases/sub-task | Min. requirement sections |
|-------------|--------------------:|-------------------------:|--------------------------:|
| XS / S      | 20+                 | 1                        | 2                         |
| M           | 40+                 | 2                        | 3                         |
| L / XL      | 60+                 | 3                        | 4+                        |

Simple tickets (no sub-tasks, XS/S) only need a **Quick-Check**: verify that requirements are
unambiguous and edge cases are covered. Skip sections that don't apply (e.g., no API section
for a pure frontend bugfix).

---

## Data Model

- [ ] Schema fully defined (all columns, types, nullable, defaults)
- [ ] Indexes specified (performance-relevant queries covered)
- [ ] Foreign keys defined (incl. ON DELETE behavior)
- [ ] Soft-delete considered (if project uses it — check CLAUDE.md)
- [ ] Unique constraints specified (where business logic requires it)
- [ ] Seed data / initial data defined (if needed)

## UX / UI

- [ ] ASCII mockup present (for each UI component)
- [ ] Icon mapping defined (referencing the project's icon library)
- [ ] Responsive behavior described (mobile / tablet / desktop)
- [ ] Interaction states complete:
  - [ ] Empty state (no data available)
  - [ ] Loading state (data loading)
  - [ ] Error state (load/save failure)
  - [ ] Success state (normal display)
- [ ] Dark/light mode considered
- [ ] Keyboard navigation specified (where relevant)
- [ ] Animations / transitions described (where relevant)

## API

- [ ] Endpoints fully defined (path, HTTP method, auth)
- [ ] Request DTOs specified (all fields, types, constraints)
- [ ] Response DTOs specified (all fields, types)
- [ ] Error responses defined (HTTP status codes, error messages)
- [ ] Pagination specified (for list endpoints)
- [ ] Authorization rules clear (which roles/claims)

## Validation

- [ ] Validation rules per field defined (min/max, format, required)
- [ ] Error messages specified (user-facing text)
- [ ] Client-side vs. server-side validation clearly separated
- [ ] Validation approach matches project conventions (check CLAUDE.md)

## Integration

- [ ] Existing systems/components identified (reuse vs. new)
- [ ] API spec update considered
- [ ] Side effects on other features documented
- [ ] Backward compatibility assessed

## Edge Cases

- [ ] Edge cases per sub-task identified (count per thresholds above)
- [ ] Concurrent access considered (where relevant)
- [ ] Deletion of referenced entities handled
- [ ] Limits defined (max lengths, max counts)

## Sub-Task Quality

- [ ] Each sub-task meets minimum line count for its size (see thresholds above)
- [ ] Each sub-task has the minimum number of requirement sections (see thresholds above)
- [ ] Each sub-task has: Goal, Dependencies, Requirements, Estimated Size, Notes
- [ ] Dependencies between sub-tasks are correct and complete
- [ ] No circular dependencies
- [ ] Implementation order is logical (data model before endpoints before frontend)
