# Example Workflow Overrides

Skills support free-form overrides via a `Workflow Overrides` section in CLAUDE.md.
This file shows common override patterns organized by skill.

---

## How Overrides Work

Add a `## Workflow Overrides` section to your CLAUDE.md with natural-language rules.
Skills read this section and adjust their behavior accordingly.

```markdown
## Workflow Overrides
- {skill}: {what to change}
```

---

## Implementation Overrides

```markdown
## Workflow Overrides

# Always run linting after changes
- Implementation: Run `npm run lint` after every file change

# Skip tests for trivial changes
- Implementation: No tests required for CSS-only or documentation changes

# Enforce specific tools
- Implementation: Always use Zod for validation, never manual checks
- Implementation: Use TanStack Query for all API calls, never raw fetch

# Limit scope
- Implementation: Never modify files outside the feature directory without asking first
- Implementation: Maximum 5 files changed per implementation — split into multiple PRs if larger
```

---

## Code Review Overrides

```markdown
## Workflow Overrides

# Skip sections that don't apply
- Code review: Skip performance section for prototype branches
- Code review: Don't flag missing JSDoc in internal utility functions

# Add project-specific checks
- Code review: Verify all new API endpoints have rate limiting middleware
- Code review: Check that Zod schemas are defined in shared package, not duplicated
- Code review: Ensure all database queries use the repository pattern

# Adjust strictness
- Code review: Treat console.log in production code as a blocking issue
- Code review: Allow magic numbers in test files
```

---

## Commit Overrides

```markdown
## Workflow Overrides

# Ticket references
- Commits: Always include ticket ID in scope (e.g., feat(PROJ-42): ...)
- Commits: Add "Closes PROJ-42" in commit body for completed tickets

# Multi-repo behavior
- Commits: Suggest separate commits per repository
- Commits: Group related changes across packages into one commit

# Message style
- Commits: Use imperative mood in descriptions
- Commits: Keep first line under 60 characters
```

---

## Ticket Overrides

```markdown
## Workflow Overrides

# Language
- Tickets: Write ticket descriptions in German
- Tickets: Keep technical terms in English

# Structure
- Tickets: Always include an "Impact Assessment" section
- Tickets: Add "Rollback Plan" for infrastructure changes
- Tickets: Include estimated story points (XS=1, S=2, M=3, L=5, XL=8)

# Refinement depth
- Refine: Include ASCII mockups for every UI sub-task
- Refine: Always specify mobile responsive behavior
- Refine: Include OpenAPI spec snippets for API endpoints
```

---

## Investigation Overrides

```markdown
## Workflow Overrides

# Default time ranges
- Investigate: Default to last 6 hours instead of last 1 hour
- Investigate: Always check both API and web service logs

# Alert response
- Investigate: When checking disk space, also check Docker image sizes
- Investigate: Include container restart count in every investigation
```

---

## Combined Example

A realistic CLAUDE.md section combining multiple overrides:

```markdown
## Workflow Overrides
- Implementation: Always run `pnpm lint` and `pnpm typecheck` after changes
- Implementation: No tests required for CSS-only changes
- Code review: Check that API responses follow RFC 7807 error format
- Code review: Verify new endpoints have integration tests
- Commits: Include ticket ID in scope
- Commits: Separate commits for API and frontend changes
- Tickets: Include story point estimates
- Refine: ASCII mockups required for all UI sub-tasks
```
