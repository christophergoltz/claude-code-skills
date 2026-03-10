# Code Review Checklist

Reference checklist for the `code-review` skill. Each category lists specific items to
check when reviewing code changes.

Adapt this checklist to the project's tech stack. Check CLAUDE.md for project-specific
patterns and conventions that should be added to or override items here.

---

## 1. Compiler & Warnings

- [ ] Project builds without errors
- [ ] No new warnings introduced by the change
- [ ] No type-safety warnings (nullability, type mismatch, etc.)

## 2. SOLID Principles

### Single Responsibility (SRP)
- [ ] Each class has one clear responsibility
- [ ] Each method does one thing
- [ ] No "god methods" that handle multiple concerns (validation + business logic + persistence)

### Open/Closed (OCP)
- [ ] New behavior added via extension, not by modifying existing working code
- [ ] Switch statements on types → consider polymorphism (only if 3+ cases)

### Liskov Substitution (LSP)
- [ ] Derived classes honor base class contracts
- [ ] No `NotImplementedException` / `throw new Error("not implemented")` in interface implementations

### Interface Segregation (ISP)
- [ ] Interfaces are focused (not "kitchen sink" interfaces)
- [ ] No empty method implementations to satisfy an interface

### Dependency Inversion (DIP)
- [ ] Dependencies injected, not created with `new` in business logic
- [ ] Depends on abstractions, not concrete classes
- [ ] No service locator pattern in business logic

## 3. Clean Code

- [ ] Variable/method/class names are descriptive and self-documenting
- [ ] No single-letter variable names (except `i`/`j` in short loops, `x` in simple lambdas)
- [ ] No magic numbers or strings — extract into named constants
  - Numeric literals (other than `0`, `1`, `-1`) must be constants
  - String literals used for logic (keys, roles, error codes, route segments) must be constants
  - Repeated string literals (same value 2+ times) must be a single shared constant
  - Acceptable exceptions: framework-specific config DSLs, log message templates
- [ ] Methods are short and focused (guideline: < 80 lines)
- [ ] No commented-out code
- [ ] No redundant comments that repeat what the code says
- [ ] Consistent formatting within the changed files

## 4. DRY / KISS / YAGNI

### DRY (Don't Repeat Yourself)
- [ ] No duplicated code blocks (3+ lines identical or near-identical)
- [ ] No copy-paste with minor variations → extract shared logic

### KISS (Keep It Simple)
- [ ] No over-engineered abstractions for simple operations
- [ ] No unnecessary design patterns (factory for a single type, strategy for one algorithm)
- [ ] Conditional logic is straightforward (no deeply nested if/else)

### YAGNI (You Aren't Gonna Need It)
- [ ] No unused parameters, methods, or properties
- [ ] No speculative generalization ("might need this later")
- [ ] No feature flags or configuration for things that have one value
- [ ] No empty catch blocks or placeholder methods

## 5. Error Handling

- [ ] Expected failure cases use result types or explicit error handling (not exceptions for control flow)
- [ ] Null/undefined checks on external inputs (API parameters, database results)
- [ ] No generic catch-all exception handlers without re-throw or specific handling
- [ ] Stack traces preserved when re-throwing exceptions
- [ ] No swallowed exceptions (empty catch blocks)
- [ ] Validation errors return appropriate HTTP status codes (400, not 500)

## 6. Project Patterns

Check CLAUDE.md and the existing codebase for project-specific patterns. Common things
to verify:

- [ ] New code follows the project's established architectural patterns
- [ ] Naming conventions match the project's style
- [ ] Error handling matches the project's approach (Result types, exceptions, error codes, etc.)
- [ ] Mapping/serialization follows project conventions
- [ ] API endpoint patterns match existing endpoints

## 7. Security

- [ ] API endpoints have appropriate authorization checks
- [ ] User ownership/access verified (not just "is authenticated" but "can access this resource")
- [ ] No sensitive data logged (passwords, tokens, personal data)
- [ ] Input validated before processing
- [ ] No raw SQL queries or string interpolation in queries (use parameterized queries)
- [ ] No unsafe HTML rendering without sanitization
- [ ] No hardcoded secrets, connection strings, or API keys

## 8. Performance

### Database / Data Access
- [ ] Filters applied before data materialization (not filter-after-fetch)
- [ ] No N+1 queries (use eager loading for needed related data)
- [ ] Read-only queries use read-only mode where available (e.g. `.AsNoTracking()`)
- [ ] Pagination used for list endpoints (no unbounded queries)

### Async / Concurrency
- [ ] No blocking on async calls (deadlock risk)
- [ ] No fire-and-forget async without explicit intent
- [ ] Cancellation tokens forwarded to I/O operations where available

### General
- [ ] No string concatenation in loops → use appropriate builder/join
- [ ] No repeated expensive operations in loops (DB calls, HTTP requests)
