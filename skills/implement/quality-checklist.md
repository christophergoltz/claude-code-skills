# Implementation Quality Checklist

Self-review checklist for the `implement` skill. Use this during Phase 4 to ensure
production-ready code quality. Each item describes what to DO, not what to check.

Adapt this checklist to the project's tech stack. Check CLAUDE.md for project-specific
patterns and conventions that should be added to or override items here.

---

## 1. Compiler & Warnings

- Ensure the project builds without errors after every batch of changes
- Use IDE MCP tools for instant feedback after individual file changes (if available)
- Avoid introducing type-safety warnings (nullability, type mismatch, etc.)
- Resolve any new compiler warnings before moving on

## 2. SOLID Principles

### Single Responsibility (SRP)
- Give each class one clear responsibility
- Make each method do one thing
- Separate concerns: don't mix validation + business logic + persistence in one method

### Open/Closed (OCP)
- Add new behavior via extension, not by modifying existing working code
- Consider polymorphism over switch statements on types (when 3+ cases)

### Liskov Substitution (LSP)
- Honor base class contracts in derived classes
- Never leave unimplemented interface methods (e.g. `NotImplementedException`, `throw new Error`)

### Interface Segregation (ISP)
- Keep interfaces focused — avoid "kitchen sink" interfaces
- Don't add empty method implementations just to satisfy an interface

### Dependency Inversion (DIP)
- Inject dependencies, don't create them with `new` in business logic
- Depend on abstractions, not concrete classes
- Avoid service locator pattern in business logic

## 3. Clean Code

- Use descriptive, self-documenting names for variables, methods, and classes
- Avoid single-letter variable names (except `i`/`j` in short loops, `x` in simple lambdas)
- Extract magic numbers and strings into named constants
  - Use constants for numeric literals other than `0`, `1`, `-1`
  - Use constants for string literals used in logic (keys, roles, error codes, route segments)
  - When the same string literal appears 2+ times, consolidate into a single shared constant
  - Acceptable without constants: framework-specific config DSLs, log message templates
- Keep methods short and focused (guideline: < 80 lines)
- Remove commented-out code — don't leave dead code behind
- Don't add comments that just repeat what the code says
- Maintain consistent formatting within modified files

## 4. DRY / KISS / YAGNI

### DRY (Don't Repeat Yourself)
- Extract duplicated code blocks (3+ lines identical or near-identical) into methods
- Don't copy-paste with minor variations — extract and parameterize

### KISS (Keep It Simple)
- Don't create abstractions for simple operations
- Don't apply design patterns unnecessarily (no factory for a single type, no strategy for one algorithm)
- Keep conditional logic straightforward — avoid deeply nested if/else

### YAGNI (You Aren't Gonna Need It)
- Don't add unused parameters, methods, or properties
- Don't generalize speculatively ("might need this later")
- Don't add feature flags or configuration for things with only one value
- Don't leave empty catch blocks or placeholder methods

## 5. Error Handling

- Use the project's established error handling pattern (Result types, exceptions, error codes, etc.)
- Add null/undefined checks on external inputs (API parameters, database results, user input)
- Don't use generic catch-all exception handlers without re-throw or specific handling
- Preserve stack traces when re-throwing exceptions
- Never swallow exceptions with empty catch blocks
- Return appropriate HTTP status codes for validation errors (400, not 500)

## 6. Project Patterns

Check CLAUDE.md and the existing codebase for project-specific patterns. Common things
to follow:

- Endpoint/route registration patterns
- Error/result handling conventions
- Input validation approach
- Object mapping conventions
- Entity/model conventions (immutability, audit fields, soft delete)
- Naming conventions (async suffix, DTO naming, etc.)
- Component communication patterns (events, callbacks, state management)
- API client patterns (code generation, manual clients, etc.)

## 7. Security

- Add appropriate authorization checks to all endpoints/routes
- Verify user ownership/access — not just "is authenticated" but "can access this resource"
- Don't log sensitive data (passwords, tokens, personal data)
- Validate input before processing
- Use parameterized queries — no raw SQL or string interpolation in queries
- No unsafe HTML rendering without sanitization
- Never hardcode secrets, connection strings, or API keys

## 8. Performance

### Data Access
- Apply filters before data materialization (not filter-after-fetch)
- Avoid N+1 queries (use eager loading for needed related data)
- Use read-only mode for read-only queries where available
- Use pagination for list endpoints — no unbounded queries

### Async / Concurrency
- No blocking on async calls (deadlock risk)
- No fire-and-forget async without explicit intent
- Forward cancellation tokens to I/O operations where available

### General
- Use appropriate string builder/join instead of string concatenation in loops
- No repeated expensive operations in loops (DB calls, HTTP requests)
