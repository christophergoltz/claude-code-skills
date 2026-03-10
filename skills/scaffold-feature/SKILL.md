---
name: scaffold-feature
description: >
  Generate a complete new feature from scratch by scaffolding all boilerplate files following
  the project's established patterns. Uses a "Scaffold by Example" approach — reads an existing
  feature in the codebase as a reference and replicates its structure for the new feature.
  Use when the user says "scaffold", "new feature", "generate boilerplate", "new feature slice",
  "CRUD endpoint", or describes wanting all standard files for a new entity/feature.
  Do NOT use for modifying existing features (use implement), fixing bugs, or creating tickets.
argument-hint: "[optional: feature name, e.g. 'Products']"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Scaffold Feature Skill

You are generating a complete feature scaffold by reading an **existing feature in the codebase**
as a reference and replicating its structure for a new feature. This is the "Scaffold by Example"
approach — the codebase IS the template.

Read the project's CLAUDE.md to understand:
- Project structure and architecture patterns
- Build commands
- Naming conventions
- Database migration approach

**Communicate in the user's language.** Detect from their messages and respond accordingly.

## CLAUDE.md Requirements

Before starting, read CLAUDE.md and verify these are documented. If **required** info is
missing, use `AskUserQuestion` to ask the user to provide it before proceeding.

| Info | Required | Used for |
|------|----------|----------|
| Build commands | **Yes** | Build verification after generation |
| Project structure | Recommended | Knowing where features live, finding reference features |
| Naming conventions | Recommended | Consistent naming in generated files |
| DB migration approach | Recommended | Reminding user of manual migration steps |

## Important Principles

1. **Scaffold by Example**: ALWAYS read an existing feature as reference before generating.
   The codebase is the template — never generate patterns from memory alone.
2. **Present plan before writing**: Show all files that will be generated and get user confirmation
   before writing anything.
3. **Minimal scope**: Only generate the standard boilerplate. Do not add business logic,
   special endpoints, or extra features beyond what the user requests.
4. **Build verification**: ALWAYS run the build command after generation to verify correctness.
5. **No database migrations**: NEVER generate database migrations — remind the user to do this
   manually using the project's migration approach.

## Process

### Step 1: Gather Requirements

Determine from the user (or `$ARGUMENTS`):
- **Feature name** (plural, PascalCase, e.g., `Products`) — used for folder, class prefixes, route
- **Entity/model name** (singular, PascalCase, e.g., `Product`) — used for model class, DTOs
- **Properties**: What fields does this entity have beyond inherited/base fields?
- **CRUD scope**: Which operations? Default: full CRUD (List, GetById, Create, Update, Delete)

If the user provides just a name, ask about the entity-specific properties.

### Step 2: Find and Read Reference Feature

**This is the critical step.** Find an existing feature in the codebase to use as reference:

1. Use `Glob` to find feature directories:
   ```
   **/Features/*/
   **/features/*/
   **/modules/*/
   ```

2. If multiple features exist, choose the one most similar to the new feature
   (similar entity shape, similar CRUD operations).

3. Read ALL files in the reference feature to understand the complete pattern:
   - Endpoints/Controllers/Routes
   - Services/Handlers
   - Queries/Repositories
   - DTOs/Request-Response models
   - Validators
   - Mappers
   - Interfaces/Abstractions
   - Tests

4. Also check:
   - How services are registered (DI container configuration)
   - How routes are registered
   - Base classes or interfaces that new features must implement
   - Naming conventions used across all files

### Step 3: Present Plan

Show the user exactly which files will be generated, mirroring the reference structure:

```
Reference feature: {ReferenceFeatureName}/
New feature:       {NewFeatureName}/

Files to generate:
{NewFeatureName}/
├── {file1}
├── {file2}
├── SubDir/
│   ├── {file3}
│   └── {file4}
└── ...

Modified files:
  {DI registration file}
  {Route registration file (if separate)}
```

Also list manual steps the user will need to do after:
- Database migration (if applicable)
- Any other registration/configuration not automated

Wait for user confirmation before proceeding.

### Step 4: Generate Files

For each file in the reference feature:
1. Read the reference file
2. Replace the reference feature/entity names with the new names
3. Adapt properties to match the new entity
4. Keep all patterns, conventions, and structure identical
5. Write the new file

**Key replacements** (adapt case variants):
- `ReferenceFeature` → `NewFeature` (plural, PascalCase)
- `ReferenceEntity` → `NewEntity` (singular, PascalCase)
- `referenceFeature` → `newFeature` (plural, camelCase)
- `referenceEntity` → `newEntity` (singular, camelCase)
- `reference-feature` → `new-feature` (plural, kebab-case — for routes)
- `reference-entity` → `new-entity` (singular, kebab-case)

### Step 5: Register Services / Routes

Update the project's DI configuration and/or route registration to include the new feature.
Follow the exact same pattern used by the reference feature.

### Step 6: Build Verification

Run the project's build command (from CLAUDE.md):

```bash
# Example — use the actual build command from CLAUDE.md
dotnet build <SolutionFile>
# or: npm run build
# or: cargo build
# etc.
```

If build fails, fix the errors. Common issues:
- Missing imports/using statements
- Typos in type names
- Missing properties in mapper
- Missing DI registration

### Step 7: Remind User of Manual Steps

After successful build, remind the user of any manual steps:

1. **Database migration** (if the entity has a database table):
   - Describe how to create the migration using the project's approach
2. **Seed/demo data** (optional): If the project has seed data, mention it
3. **Tests**: Mention that tests should be added following the reference feature's test pattern

### Step 8: Commit Message Suggestions

Suggest commit messages based on `git status`. Use Conventional Commits format (English).
Do NOT commit — only suggest. Git is read-only.

## Rules

- NEVER generate database migrations — remind the user to do this manually.
- NEVER modify existing features or shared infrastructure (except DI/route registration).
- ALWAYS read the reference feature before generating — never generate from memory alone.
- ALWAYS present the file list and get confirmation before writing.
- ALWAYS run the build command after generation.
- ALWAYS register new services/routes following the reference pattern.
- Generated code must compile without errors (fix if not).
- Git is read-only — NEVER run write-mode git commands.
